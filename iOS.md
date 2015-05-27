> 1. Read the documentation.
> 2. Read the documentation.
> 3. Remember that you were warned twice about reading the documentation.

===
多使用字面量语法，但有限制：除了字符串以外，所创建出来的对象必须属于`Foundation`框架。  
使用字面量语法创建的对象都是不可变的(immutable)。

===
少用`#define`预处理命令，多用类型常量(带类型信息)。
```objective-c
    // Use：
    static const NSTimeInterval kAnimationDuration = 0.3;
    // Deprecated
    #define ANIMATION_DURATION 0.3
```

===
若常量局限于实现文件( .m )之内，则在前面加字母`k`；  
若在类之外可见，则以类名为前缀。

===
全局符号表(global symbol table)：全局常量必须定义且只能定义一次
```objective-c
    // In the header file
        extern NSString *const AGStringConstant;
    // In the implementation file
        NSString *const AGStringConstant = @"value";
```

===
多用枚举(`NS_ENUM`, `NS_OPTIONS`)表示状态、选项、状态码。

===
`self.xxx` : 通过属性访问,  `_xxx` : 直接访问实例变量。  
`_xxx` 直接操作指针,  `self.xxx` 调用了`setter`方法操作 `_xxx` 指针。  
`_xxx` VS `self.xxx` ：
- 决不在`init`或`dealloc`方法中调用`setter/getter`方法(不用 `self.xxx`，使用`_xxx`)，因为子类可能会覆盖掉该方法 。
- `_xxx` 不经过OC的 method dispatch 步骤，直接访问变量的内存，所以速度较快。
- `_xxx` 不会调用`setter`方法，绕过了为相关属性所定义的“内存管理语义”。
- `_xxx` 不会触发 Key-value Observing。
- `self.xxx` 有助于排查相关错误，因为可以在`setter/getter`方法中设断点。
- 如果使用了“惰性初始化”(lazy initialization)技术，必须使用 `self.xxx`。

**因此，写入时使用 `self.xxx` ，读取时使用 `_xxx` 。**

===
iOS使用同步锁的开销较大，会严重影响性能，所以一般使用`nonatomic`属性，但Mac OS X使用`atomic`属性通常不会有性能瓶颈。

===
相同的对象必须具有相同的哈希码，但哈希码相同的对象却未必相同。

===
iOS查看crash log:

1. 拿到crash文件（可以通过itunes同步，并在`~/Library/Logs/CrashReporter/MobileDevice`下拿到相关crash文件）
2. 拿到`xxx.app`以及`xxx.app.dSYM`文件（直接选中Xcode的.app文件右击show in finder找到）
3. 找到symbolicatecrash文件放到和 2 的文件相同目录(通过命令找到：`find /Applications/Xcode.app -name symbolicatecrash -type f` )
4. `export DEVELOPER_DIR="/Applications/XCode.app/Contents/Developer"`
5. 切换到app文件夹下执行如下命令：`./symbolicatecrash xxx_2015-01-04-145029_Xxx-iPhone6.crash xxx.app.dSYM > iphone6.crash`
6. 这样就拿到了iphone6.crash的转换后的文件了。

===
视图的生命周期:
```objective-c
init  ->  loadView  ->  viewDidLoad  ->  viewWillAppear  ->  viewDidAppear  ->
-> viewWillDisappear  ->  viewDidDisappear  - (此时视图已消失了；当模拟内存警告的时候) ->
-> viewWillUnload  ->  viewDidUnload
```
视图控制对象通过`alloc`和`init`来创建，但是视图控制对象不会在创建的那一刻就马上创建相应的视图，而是等到需要使用的时候才通过调用`loadView`来创建.

===
- 什么时候用`assign`: 
 - 基础类型 (简单类型，原子类型): `NSInteger`, `CGPoint`, `CGFloat`
 - C数据类型: `int`, `float`, `double`, `char`等
- 什么时候用`copy`: 
 - 含有可深拷贝的mutable子类的类，如`NSArray`, `NSSet`, `NSDictionary`, `NSData`, `NSCharacterSet`, `NSIndexSet`, `NSString`..
 - **但是`NSMutableArray`这样的不可以，Mutable的不能用`copy`,不然初始化会有问题，切记。**
- 什么时候用`retain`: 
 - 其他`NSObject`和其子类对象(大多数)。

===
对于直接从xib或者storyboard拉出来生成的`IBOutlet`属性，是选择`strong`还是`weak`呢？
- 如果该控件位于控件树的顶部，比如 `UIViewController下的view`，那就应该选择`strong`，因为`viewController`直接拥有该`view`。
- 如果控件是`viewController`中`view`的子视图，对于这个子视图，它的所有者是它的父视图，代码中只是想引用一下这个子视图的指针而已，那么就应该选择`weak`.

===
对于`NSArray`和`NSDictionary`通过**[]**获取到的对象是`id`类型，因为[index]相当于调用了`NSArray`对象的`objectAtIndex`方法，[key]相当于调用了`objectForKey`，这两个方法返回的都是`id`类型的对象。想要再访问指定对象类型的方法或者成员变量还需要自己转一下。

===
`UIImageView`使用`[UIImage imageNamed:]`方法访问显示一个图片，系统会自动将此图片在内存中生成一份缓存，如果不从内存中清除此缓存，它将一直占用内存。如果使用**`imageWithContentsOfFile`**，当`UIImageView`由一张图片显示为另一张图片的时候，系统就会自动将上一张图片从内存中销毁移除。

===
`UITextField`左右两侧有`leftView`和`rightView`。如果我们想在用户输入的时候自动在左侧留有空隙，那么就可以使用指定`leftView`为`[[UIView alloc] initWithRect:CGRectMake(0, 0, 8, 0)]`;

===
**`@()`**，是将常见数据类型临时变量转换为指定OC类型。比如`int a = 20;NSNumber *aa = @(a)`;

===
OC中所以属性都是以零/`nil`开始的.

===
给`nil`发送消息总是返回0(`NO`)。  
`[nil isEqual: nil]` 返回`NO`。

===
`strong`，`weak` 用来修饰**属性**。  
`__weak`, `__strong` 还有 `__unsafe_unretained`, `__autoreleasing` 都是用来修饰**变量**。  
`__strong` 是缺省的关键词。  
`__weak` 声明了一个可以自动`nil`化的弱引用，弱引用：不增加保留计数；  
`__unsafe_unretained` 声明弱引用，但不自动`nil`化，若所指内存区域被释放，这指针就变野指针了。  
`__autoreleasing` 用来修饰一个函数的参数，这个参数会在函数返回的时候被自动释放。

===
内省（Introspection）是对象揭示自己作为一个运行时对象的详细信息的一种能力。这些详细信息包括对象在继承树上的位置，对象是否遵循特定的协议，以及是否可以响应特定的消息。`NSObject`协议和类定义了很多内省方法，用于查询运行时信息，以便根据对象的特征进行识别:
- `isKindOfClass:Class`  - 检查对象是否是那个类或者其继承类实例化的对象
- `isMemberOfClass:Class`  - 检查对象是否是那个类但不包括继承类而实例化的对象
- `respondToSelector:selector`  - 检查对象是否包含这个方法
- `conformsToProtocol:protocol`  - 检查对象是否符合协议，是否实现了协议中所有的必选方法。

===
`controller`的首要任务是同步模型，UI；

===
ARC与垃圾回收器不同，不在运行时工作而是编译时工作，在代码中自动插入合适的`retain`，`release`。

===
属性名称不能以 `new` 开头；

===
只要对象实现了委托方法，任何类的对象都可成为委托对象；

===
`UIView`: Never call `drallRect`, use `setNeedsDisplay`;

===
随机数：`arc4random()%X ` ： 0 ~ X-1；

===
OC不存在真正的私有方法；

===
任何C源程序不经修改即可通过OC汇编器成功编译；

===
所有对象都使用动态分配的内存；

===
`@interface`有且只有一个实现；

===
所有对象的交互都通过指针实现；

===
继承：isa；
复合：has a，对象之间的组合才叫复合；

===
`#import`本质上只进行了剪切和粘贴操作；

===
点操作符只用于`setter`，`getter`方法；

===
**-** 实例方法；  
**+** 类方法；

===
OC单一继承；

===
`[NSNull null]` 方法始终都会返回相同的实例。按此方式工作的类称为***单例类***。

===
`@class`创建向前引用；

===
一定不要直接调用`dealloc`方法，OC会在需要销毁对象时自动调用；

===
`id`很像`void*`，但被严格限制只能使用在对象上；

===
不要使用任何刚释放的内存，否则会误用陈旧的数据；

===
**`blocks`是对象；**

===
`NSDictionary`，`NSArray`只能储存OC对象；

===
`@protocol`：一系列不属于任何类的方法，可以被任何类实现，在不同场景采用不同的实现，如iOS和OS X。

===
`@category`：无法向类中添加实例变量,  可在类别(`@category`)中添加属性(`@property`)，且必须是`@dynamic`类型的；当发生名称冲突时，`category`具有更高的优先级；

===
`UIButton`如果要设置image，title colour，title等值的时候，**必须要指定一个值给normal state的时候，而在其他状态的时候可以不用指定一个值**，也就是为什么一定要用
```objective-c
- (void)setTitle:(NSString *)title forState:(UIControlState)state; 
```
这个方法来设定`UIButton`的title了，主要是给`UIControlStateNormal`状态是设置一个指定的值，而`textLabel.text = @“xx”` 是无效的

===
`UIApplication`是单例模式，一个应用程序只有一个`UIApplication`对象或子对象；

===
调用 `UIApplicationMain` 会创建应用程序的两个重要初始组件：
- [`UIApplication` 类的实例](https://developer.apple.com/library/prerelease/ios/documentation/UIKit/Reference/UIApplication_Class/index.html#//apple_ref/occ/cl/UIApplication)，称为应用程序对象。
应用程序对象可管理应用程序事件循环，并协调其他高级的应用程序行为。定义在 `UIKit` 框架中的这个类，不要求您编写任何额外的代码，就可以达成其任务。
- `AppDelegate` 类的实例，称为应用程序委托。
Xcode 创建此类，作为设置 `Application` 模板的一部分。应用程序委托会创建一个呈现应用程序内容的窗口，并为响应应用程序内的状态转换提供位置。这个窗口是您编写自定应用程序级代码的地方。

===
应用程序对象和应用程序委托互动的方式：应用程序启动时，应用程序对象会调用应用程序委托上已定义的方法，使自定代码有机会执行其操作(当`UIApplication`运行过程中引发了某个事件之后会调用代理中对应的方法)。应用程序委托的界面包含了单一属性：`window`。有了这个属性，应用程序委托才会跟踪能呈现所有应用程序内容的窗口。

===
`sender` 参数指向负责触发操作的对象。

===
将由特定导航控制器(`UINavigationController`)所管理的一组视图控制器称为其导航栈。导航栈是一组后进先出的自定视图控制器对象。添加到堆栈的第一个项目将变成根视图控制器，永不会从堆栈中弹出。而其他视图控制器可被压入或弹出导航栈。

===
可以创建以下几种类型的过渡(`segue`)：
- **Push**: Push过渡将目的视图控制器添加到导航栈。只有当源视图控制器与导航控制器连接时，才可以使用Push过渡。
- **Modal**: 简单而言，Modal过渡就是一个视图控制器以模态方式显示另一个控制器，需要用户在显示的控制器上执行某种操作，然后返回到应用程序的主流程。Modal
视图控制器不会添加到导航栈；相反，它通常被认为是所显示的视图控制器的子视图控制器。显示的视图控制器的作用是关闭它所创建和显示的 Modal 视图控制器。
- **Custom**: 可以通过将 `UIStoryboardSegue` 子类化来定义自定过渡。
- **Unwind**: Unwind过渡通过向后移动一个或多个过渡，让用户返回到视图控制器的当前实例。使用Unwind过渡可以实现反向导航。

除了过渡之外，还可以通过关系来连接场景。例如，导航控制器与其根视图控制器之间就存在关系。就此而言，这种关系表示导航控制器包含根视图控制器。

===
委托是一种简单而强大的模式。在此模式中，应用程序中的一个对象代表另一个对象，或与另一个对象协调工作。授权对象保留对另一个对象（委托对象）的引用，并适时向委托对象发送信息。该信息会告诉事件的委托对象，授权对象即将处理或刚处理了某个事件。委托对象可能会对该信息作出如下响应：更新其本身或应用程序中其他对象的外观或状态，在某些情况下，它会返回一个值来反映待处理的事件该如何处理。

===
字典 (`NSDictionary`) 会储存与给定键相关的对象，用于以后的检索。最佳实践是将字符串对象用作字典键。虽然其他对象也可以用作键，但要注意，每个键都会被拷贝以供字典使用，并且必须支持 `NSCopying`。不过，如果要使用键－值编码，则必须为字典对象使用字符串键。若要了解更多信息，请参阅《[Key-Value Coding Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/KeyValueCoding.html#//apple_ref/doc/uid/10000107i)》（键值编码编程指南）。

===
对于字典字面常量，键会在其对象前被指定，并且对象列表和键不以 `nil` 结束。

===
`NSSet`，`NSMutableSet`类声明编程接口对象，无序的集合，在内存中存储方式是不连续的，不像`NSArray`，`NSDictionary`（都是有序的集合）类声明编程接口对象是有序集合，在内存中存储位置是连续的；

===
属性可以是私有的或公共的。有时让属性私有是合理的选择，这样其他类就不能查看或访问它，方法是将它放在实现文件顶部的***类扩展***中。

===
`NSUserDefaults` 存储的对象全是不可变的(immutable)，取出数据也是一样的。

===
只要一个类返回自身的实例，用`instancetype`就有好处。  
对于简易构造函数(convenience constructor)，应该总是用`instancetype`。编译器不会自动将`id`转化为`instancetype`。`id`是通用对象，但如果你用`instancetype`，编译器就知道方法返回什么类型的对象。

===
隐藏键盘: `[self.view endEditing : YES]`

===
`UIScrollView`的属性总结:

属性	| 作用
---- | ----
`CGPoint contentOffSet` | 监控目前滚动的位置
`CGSize contentSize` | 滚动范围的大小
`UIEdgeInsets contentInset` | 视图在`scrollView`中的位置
`id<UIScrollerViewDelegate>delegate` | 设置协议
`BOOL directionalLockEnabled` | 指定控件是否只能在一个方向上滚动
`BOOL bounces` | 控制控件遇到边框是否反弹
`BOOL alwaysBounceVertical` | 控制垂直方向遇到边框是否反弹
`BOOL alwaysBounceHorizontal` | 控制水平方向遇到边框是否反弹
`BOOL pagingEnabled` | 控制控件是否整页翻动
`BOOL scrollEnabled` | 控制控件是否能滚动
`BOOL showsHorizontalScrollIndicator` | 控制是否显示水平方向的滚动条
`BOOL showsVerticalScrollIndicator` | 控制是否显示垂直方向的滚动条
`UIEdgeInsets scrollIndicatorInsets` | 指定滚动条在`scrollerView`中的位置
`UIScrollViewIndicatorStyle indicatorStyle` | 设定滚动条的样式
`float decelerationRate` | 改变`scrollerView`的减速点位置
`BOOL tracking` | 监控当前目标是否正在被跟踪
`BOOL dragging` | 监控当前目标是否正在被拖拽
`BOOL decelerating` | 监控当前目标是否正在减速
`BOOL delaysContentTouches` | 控制视图是否延时调用开始滚动的方法
`BOOL canCancelContentTouches` | 控制控件是否接触取消`touch`的事件
`float minimumZoomScale` | 缩小的最小比例
`float maximumZoomScale` | 放大的最大比例
`float zoomScale` | 设置变化比例
`BOOL bouncesZoom` | 控制缩放的时候是否会反弹
`BOOL zooming` | 判断控件的大小是否正在改变
`BOOL zoomBouncing` | 判断是否正在进行缩放反弹
`BOOL scrollsToTop` | 控制控件滚动到顶部

===
`UIScrollView`的几个要点总结：

从手指`touch`屏幕开始，`scrollView`会开始一个`timer`，如果：

1. 150ms内如果手指没有任何动作，消息就会传给`subView`。
2. 150ms内手指有明显的滑动（一个swipe动作），`scrollView`就会滚动，消息不会传给`subView`。
3. 150ms内手指没有滑动，`scrollView`将消息传给`subView`，但是之后手指开始滑动，`scrollView`传送`touchesCancelled`消息给`subView`，然后开始滚动。

`delaysContentTouches`的作用：  
这个标志默认是`YES`，使用上面的150ms的`timer`，如果设置为`NO`，`touch`事件立即传递给`subView`，不会有150ms的等待。

`cancelsTouches`的作用：  
这个标志默认为`YES`，如果设置为`NO`，这消息一旦传递给`subView`，这`scroll`事件不会再发生。

===
Xcode常用快捷键:

⌘ | ⌃ | ⌥ | ⇧ | ⇪ | ⊙ | ↵
---- | ---- | ---- | ---- | ---- | ---- | ----
Command | Control | Option (alt) | Shift | Caps Lock | Click | Enter

操作 | 说明
---- | ----
⌘T | 打开新标签
⌘⇧O | 快速打开文件
⌥⊙ on Symbol / 或者三根手指点击 | 快速文档，快速纲要
⌥⊙⊙ on Symbol | 打开文档窗口并定位到相关条目
⌘⊙ on Symbol  | 跳转至定义
⌃⌘↑ / ⌃⌘↓ / 三个手指垂直滑动 | 快速地在一个.h文件和对应的.m文件之间切换
⌘/ | 注释选择当前行
⌘F | 查找
⌥⌘F | 查找和替换
⇧⌘F | 在项目中查找
⌥⇧⌘F | 在项目中查找和替换
⌘: | 拼写和语法
⌥⌘F | 查找和替换
⌃I | 格式化代码
⌘↵ | 显示标准编辑器
⌥⌘↵ | 显示辅助编辑器
⌥⇧⌘↵ | 显示版本编辑器
⇧⌘Y | 显示/隐藏调试区域
⇧⌘C | 打开控制台
⌘0 | 展示/隐藏左侧导航器面板
⌘1 | 项目导航器（Project Navigator）
⌘2 | 符号导航器（Symbol Navigator）
⌘3 | 搜索导航器（Find Navigator）
⌘4 | 事件导航器（Issue Navigator）
⌘5 | 测试导航器（Test Navigator）
⌘6 | 调试导航器（Debug Navigator）
⌘7 | 断点导航器（Breakpoint Navigator）
⌘8 | 日志导航器（Log Navigator）
⌥⌘0 | 显示/隐藏右侧实用工具面板
⌥⌘1 | 文件检查器（File Inspector）
⌥⌘2 | 快速帮助（Quick Help）
⌥⌘3 | 识别检查器（Identity Inspector）
⌥⌘4 | 属性检查器（Attributes Inspector）
⌥⌘5 | 规格检查器（Size Inspector）
⌥⌘6 | 连接检查器（Connections Inspector）

===
![Image of Learning iOS](/images/Learned_route_of_iOS.jpg)

===
所有的Mac OS X和iOS程序都是由大量的对象构成，而这些对象的根对象都是`NSObject`，`NSObject`就处在`Foundation`框架之中，具体的类结构如下：

![Image of Foundation 1](/images/iOS_Foundation_class_structure_1.jpg)

![Image of Foundation 2](/images/iOS_Foundation_class_structure_2.jpg)

![Image of Foundation 3](/images/iOS_Foundation_class_structure_3.jpg)

===
`UIKit`主要用于界面构架，它的类结构：

![Image of UIKit](/images/iOS_UIKit_class_structure.jpg)