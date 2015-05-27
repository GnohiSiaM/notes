1. Read the documentation.
2. Read the documentation.
3. Remember that you were warned twice about reading the documentation.

===
多使用字面量语法，但有限制：除了字符串以外，所创建出来的对象必须属于Foundation框架。
使用字面量语法创建的对象都是不可变的(immutable)。

===
少用#define预处理命令，多用类型常量(带类型信息)。
```objective-c
    // Use：
    static const NSTimeInterval kAnimationDuration = 0.3;
    // Deprecated
    #define ANIMATION_DURATION 0.3
```

===
若常量局限于实现文件( .m )之内，则在前面加字母k；若在类之外可见，则以类名为前缀。

===
全局符号表(global symbol table)：全局常量必须定义且只能定义一次
```objective-c
    // In the header file
        extern NSString *const AGStringConstant;
    // In the implementation file
        NSString *const AGStringConstant = @"value";
```

===
多用枚举(NS_ENUM, NS_OPTIONS)表示状态、选项、状态码。

===
self.xxx : 通过属性访问,  _xxx : 直接访问实例变量。
_xxx 直接操作指针,  self.xxx 调用了setter方法操作 _xxx 指针。
_xxx VS self.xxx ：
- 决不在init或dealloc方法中调用setter/getter方法(因为子类可能会覆盖掉该方法)，不用 self.xxx ，使用 _xxx 。
- _xxx 不经过OC的 method dispatch 步骤，直接访问变量的内存，所以速度较快。
- _xxx 不会调用setter方法，绕过了为相关属性所定义的“内存管理语义”。
- _xxx 不会触发 Key-value Observing。
- self.xxx 有助于排查相关错误，因为可以在setter/getter方法中设断定。
- 如果使用了“惰性初始化”(lazy initialization)技术，必须使用 self.xxx 。

因此，写入时使用 self.xxx ，读取时使用 _xxx 。

===
iOS使用同步锁的开销较大，会严重影响性能，所以一般使用nonatomic属性，但Mac OS X使用atomic属性通常不会有性能瓶颈。

===
相同的对象必须具有相同的哈希码，但哈希码相同的对象却未必相同。

===
```objective-c
init  ->  loadView  ->  viewDidLoad  ->  viewWillAppear  ->  viewDidAppear  ->
-> viewWillDisappear  ->  viewDidDisappear  - (此时视图已消失了；当模拟内存警告的时候) ->
-> viewWillUnload  ->  viewDidUnload
```
视图控制对象通过alloc和init来创建，但是视图控制对象不会在创建的那一刻就马上创建相应的视图，而是等到需要使用的时候才通过调用loadView来创建.

===
- 什么时候用assign: 
 - 基础类型 (简单类型，原子类型): NSInteger, CGPoint, CGFloat
 - C数据类型: int, float, double, char等
- 什么时候用copy: 
 - 含有可深拷贝的mutable子类的类，如NSArray, NSSet, NSDictionary, NSData, NSCharacterSet, NSIndexSet, NSString..
 - 但是NSMutableArray这样的不可以，Mutable的不能用copy,不然初始化会有问题，切记。
- 什么时候用retain: 
 - 其他NSObject和其子类对象好嘛 (大多数)

===
对于直接从xib或者storyboard拉出来生成的IBOutlet属性，是选择strong还是weak呢？
- 如果该控件位于控件树的顶部，比如 UIViewController下的view，那就应该选择strong，因为viewController直接拥有该view。
- 如果控件是viewController中view的子视图，对于这个子视图，它的所有者是它的父视图，代码中只是想引用一下这个子视图的指针而已，那么就应该选择weak.

===
对于NSArray和NSDictionary通过[]获取到的对象是id类型，因为[index]相当于调用了NSArray对象的objectAtIndex方法，[key]相当于调用了objectForKey，这两个方法返回的都是id类型的对象。想要再访问指定对象类型的方法或者成员变量还需要自己转一下。

===
UIImageView使用[UIImage imageNamed:]方法访问显示一个图片，系统会自动将此图片在内存中生成一份缓存，如果不从内存中清除此缓存，它将一直占用内存。如果使用imageWithContentsOfFile，当UIImageView由一张图片显示为另一张图片的时候，系统就会自动将上一张图片从内存中销毁移除。

===
UITextField左右两侧有leftView和rightView。如果我们想在用户输入的时候自动在左侧留有空隙，那么就可以使用指定leftView为[[UIView alloc] initWithRect:CGRectMake(0, 0, 8, 0)];

===
@()，是将常见数据类型临时变量转换为指定OC类型。比如int a = 20;NSNumber *aa = @(a);

===
OC中所以属性都是以零/nil开始的.

===
给nil发送消息总是返回0(NO)。
[nil isEqual: nil] 返回NO。

===
strong，weak 用来修饰属性。
__weak, __strong 还有 __unsafe_unretained, __autoreleasing 都是用来修饰变量。
__strong 是缺省的关键词。
__weak 声明了一个可以自动 nil 化的弱引用，弱引用：不增加保留计数；
__unsafe_unretained 声明弱引用，但不自动nil化，若所指内存区域被释放，这指针就变野指针了。
__autoreleasing 用来修饰一个函数的参数，这个参数会在函数返回的时候被自动释放。

===
内省（Introspection）是对象揭示自己作为一个运行时对象的详细信息的一种能力。这些详细信息包括对象在继承树上的位置，对象是否遵循特定的协议，以及是否可以响应特定的消息。NSObject协议和类定义了很多内省方法，用于查询运行时信息，以便根据对象的特征进行识别:
- isKindOfClass:Class  - 检查对象是否是那个类或者其继承类实例化的对象
- isMemberOfClass:Class  - 检查对象是否是那个类但不包括继承类而实例化的对象
- respondToSelector:selector  - 检查对象是否包含这个方法
- conformsToProtocol:protocol  - 检查对象是否符合协议，是否实现了协议中所有的必选方法。

===
controller的首要任务是同步模型，UI；

===
ARC与垃圾回收器不同，不在运行时工作而是编译时工作，在代码中自动插入合适的retain，release

===
属性名称不能以 'new' 开头；

===
只要对象实现了委托方法，任何类的对象都可成为委托对象；

===
UIView: Never call 'drallRect', use 'setNeedsDisplay';

===
随机数：arc4random()%X  ： 0 ~ X-1；

===
OC不存在真正的私有方法；

===
任何C源程序不经修改即可通过OC汇编器成功编译；

===
所有对象都使用动态分配的内存；

===
@interface有且只有一个实现；

===
所有对象的交互都通过指针实现；

===
继承：isa；
复合：has a，对象之间的组合才叫复合；

===
#import本质上只进行了剪切和粘贴操作；

===
点操作符只用于setter，getter方法；

===
OC单一继承；

===
[NSNull null] 方法始终都会返回相同的实例。按此方式工作的类称为单例类。

===
@class创建向前引用；

===
一定不要直接调用dealloc方法，OC会在需要销毁对象时自动调用；

===
id很像void*，但被严格限制只能使用在对象上；

===
不要使用任何刚释放的内存，否则会误用陈旧的数据；

===
blocks是对象；

===
NSDictionary，NSArray只能储存OC对象；

===
@protocol：一系列不属于任何类的方法，可以被任何类实现，在不同场景采用不同的实现，如iOS和OS X。

===
@category：无法向类中添加实例变量,  可在类别(@category)中添加属性(@property)，且必须是@dynamic类型的；当发生名称冲突时，category具有更高的优先级；

===
UIButton如果要设置image，title colour，title等值的时候，必须要指定一个值给normal state的时候，而在其他状态的时候可以不用指定一个值，也就是为什么一定要用
```objective-c
- (void)setTitle:(NSString *)title forState:(UIControlState)state; 
```
这个方法来设定UIButton的title了，主要是给UIControlStateNormal状态是设置一个指定的值，而textLabel.text = @“xx” 是无效的

===
UIApplication是单例模式，一个应用程序只有一个UIApplication对象或子对象；

===
调用 UIApplicationMain 会创建应用程序的两个重要初始组件：
- UIApplication 类的实例，称为应用程序对象。
应用程序对象可管理应用程序事件循环，并协调其他高级的应用程序行为。定义在 UIKit 框架中的这个类，不要求您编写任何额外的代码，就可以达成其任务。
- AppDelegate 类的实例，称为应用程序委托。
Xcode 创建此类，作为设置 Application 模板的一部分。应用程序委托会创建一个呈现应用程序内容的窗口，并为响应应用程序内的状态转换提供位置。这个窗口是您编写自定应用程序级代码的地方。

===
应用程序对象和应用程序委托互动的方式：应用程序启动时，应用程序对象会调用应用程序委托上已定义的方法，使自定代码有机会执行其操作(当UIApplication运行过程中引发了某个事件之后会调用代理中对应的方法)。应用程序委托的界面包含了单一属性：window。有了这个属性，应用程序委托才会跟踪能呈现所有应用程序内容的窗口。

===
sender 参数指向负责触发操作的对象。

===
将由特定导航控制器(UINavigationController)所管理的一组视图控制器称为其导航栈。导航栈是一组后进先出的自定视图控制器对象。添加到堆栈的第一个项目将变成根视图控制器，永不会从堆栈中弹出。而其他视图控制器可被压入或弹出导航栈。

===
可以创建以下几种类型的过渡(segue)：
- Push
Push 过渡将目的视图控制器添加到导航栈。只有当源视图控制器与导航控制器连接时，才可以使用 Push 过渡。
- Modal
简单而言，Modal过渡就是一个视图控制器以模态方式显示另一个控制器，需要用户在显示的控制器上执行某种操作，然后返回到应用程序的主流程。Modal 视图控制器不会添加到导航栈；相反，它通常被认为是所显示的视图控制器的子视图控制器。显示的视图控制器的作用是关闭它所创建和显示的 Modal 视图控制器。
- Custom
可以通过将 UIStoryboardSegue 子类化来定义自定过渡。
- Unwind
Unwind 过渡通过向后移动一个或多个过渡，让用户返回到视图控制器的当前实例。使用 Unwind 过渡可以实现反向导航。

除了过渡之外，还可以通过关系来连接场景。例如，导航控制器与其根视图控制器之间就存在关系。就此而言，这种关系表示导航控制器包含根视图控制器。

===
委托是一种简单而强大的模式。在此模式中，应用程序中的一个对象代表另一个对象，或与另一个对象协调工作。授权对象保留对另一个对象（委托对象）的引用，并适时向委托对象发送信息。该信息会告诉事件的委托对象，授权对象即将处理或刚处理了某个事件。委托对象可能会对该信息作出如下响应：更新其本身或应用程序中其他对象的外观或状态，在某些情况下，它会返回一个值来反映待处理的事件该如何处理。

===
字典 (NSDictionary) 会储存与给定键相关的对象，用于以后的检索。最佳实践是将字符串对象用作字典键。虽然其他对象也可以用作键，但要注意，每个键都会被拷贝以供字典使用，并且必须支持 NSCopying。不过，如果要使用键－值编码，则必须为字典对象使用字符串键。若要了解更多信息，请参阅《Key-Value Coding Programming Guide》（键值编码编程指南）。

===
对于字典字面常量，键会在其对象前被指定，并且对象列表和键不以 nil 结束。

===
NSSet，NSMutableSet类声明编程接口对象，无序的集合，在内存中存储方式是不连续的，不像NSArray，NSDictionary（都是有序的集合）类声明编程接口对象是有序集合，在内存中存储位置是连续的；

===
属性可以是私有的或公共的。有时让属性私有是合理的选择，这样其他类就不能查看或访问它，方法是将它放在实现文件顶部的类扩展中。

