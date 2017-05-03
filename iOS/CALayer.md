与`UIView`最大的不同是`CALayer`不处理用户的交互，每一个`UIView`都有一个`CALayer`实例的图层属性，也就是所谓的 *backing layer*

`UIView`会在初始化的时候调用`+layerClass`方法，然后用它的返回类型来创建宿主图层。

`CALayer`图层才是真正用来在屏幕上显示和做动画，`UIView`仅仅是对它的一个封装，提供了一些iOS类似于处理触摸的具体功能，以及 Core Animation 底层方法的高级接口。

`CALayer`有一个属性叫做`contents`，这个属性的类型被定义为`id`，意味着它可以是任何类型的对象。
但是，在实践中，如果你给`contents`赋的不是`CGImage`，那么你得到的图层将是空白的。

如果要给图层的寄宿图赋值，按照以下这个方法： 
```swift
layer.contents = (__bridge id)image.CGImage;
```

`CALayer`与`contentMode`对应的属性叫做`contentsGravity`，**但是它是一个`NSString`类型**。

`CALayer`的`contentsScale`属性定义了寄宿图的像素尺寸和视图大小的比例，默认情况下它是一个值为1.0的浮点数。如果`contentsScale`设置为1.0，将会以每个点1个像素绘制图片，如果设置为2.0，则会以每个点2个像素绘制图片。

`UIView`有一个叫做`clipsToBounds`的属性可以用来决定是否显
示超出边界的内容，`CALayer`对应的属性叫做`masksToBounds`。

`CALayer`的`contentsRect`属性允许我们在图层边框里显示寄宿图的一个子域。  
**`contentsRect`不是按点来计算的，它使用了*单位坐标*，单位坐标指定在0到1之间，是一个相对值(像素和点就是绝对值)，所以他们是相对与寄宿图的尺寸的。**  
默认的`contentsRect`是`{0, 0, 1, 1}`，这意味着整个寄宿图默认都是可见的。

`CALayer`的`contentsCenter`是一个`CGRect`，它定义了一个固定的边框和一个在图层上可拉伸的区域。默认情况下，`contentsCenter`是`{0, 0, 1, 1}`。

给`contents`赋`CGImage`的值不是唯一的设置寄宿图的方法。也可以直接用 Core Graphics 直接绘制寄宿图。能够通过继承`UIView`并实现
`-drawRect:`方法来自定义绘制。`-drawRect:`方法没有默认的实现，因为对`UIView`来说，寄宿图并不是必须的。

**如果没有自定义绘制的任务就不要在子类中写一个空的`-drawRect:`方法。这会造成CPU资源和内存的浪费。**

`UIView`的`frame`、`bounds`和`center`属性仅仅是存取方法，当操纵这些属性时，实际上是在改变位于视图下方`CALayer`的 `frame`、`bounds`和`position`。`center`和`position`都代表了相对于父图层`anchorPoint`所在的位置。

对于视图或者图层来说，**`frame`并不是一个非常清晰的属性，它其实是一个虚拟属性**，是根据`bounds`，`position`和`transform`计算而来，所以当其中任何一个值发生改变，`frame`都会变化。

默认来说，`anchorPoint`位于图层的中点，所以图层的将会以这个点为中心放置。和`contentsRect`和`contentsCenter`属性类似，`anchorPoint`用单位坐标来描述，默认坐标是`{0.5, 0.5}`。

**和`UIView`严格的二维坐标系不同，`CALayer`存在于一个三维空间当中**。`CALayer`还有另外两个属性，`zPosition`和`anchorPointZ`，二者都是在Z轴上描述图层位置的浮点类型。

`CALayer`并不关心任何响应链事件，所以不能直接处理触摸事件或者手势。但是它有一系列的方法帮你处理事件：
`-containsPoint:`和`-hitTest:`。

`CALayer`有一个叫做`conrnerRadius`的属性控制着图层角的曲率。它是一个浮点数，默认为0。

`CALayer`另外两个非常有用属性就是`borderWidth`和`borderColor`。二者共同定义了图层边的绘制样式。这条线(也被称作stroke)沿着图层的`bounds`绘制，同时也包含图层的角。

**边框是跟随图层的边界变化的，而不是图层里面的内容。**

给`CALayer`的`shadowOpacity`属性一个大于默认值(也就是0)的值，阴影就可以显示在任意图层之下。`shadowOpacity`是一个必须在0.0(不可见)和1.0(完全不透明)之间的浮点数。  
要改动阴影的表现，可以使用`CALayer`的另外三个属性：`shadowColor`，`shadowOffset`，`shadowRadius`。  
`shadowOffset`属性控制着阴影的方向和距离。它是一个`CGSize`的值，宽度控制这阴影横向的位移，高度控制着纵向的位移。`shadowOffset`的默认值是`{0, -3}`。  
`shadowRadius`属性控制着阴影的模糊度，当它的值是0的时候，阴影就和视图一样有一个非常确定的边界线。当值越来越大的时候，边界线看上去就会越来越模糊和自然。

**和边框不同，图层的阴影继承自内容的外形，而不是根据边界和角半径来确定。**

`CALayer`有一个属性叫`mask`,这个属性本身就是个`CALayer`类型，有和其他图层一样的绘制和布局属性。不同于那些绘制在父图层中的子图层，**`mask`图层定义了父图层的部分可见区域。`mask`图层的`Color`属性是无关紧要的，真正重要的是图层的轮廓。**
蒙板图不局限于静态图。任何有图层构成的都可以作为`mask`属性，这意味着你的蒙板可以通过代码甚至是动画实时生成。

`CALayer`的`minificationFilter`和`magnificationFilter`属性。当图片需要显示不同的大小的时候，有一种叫做拉伸过滤(`kCAFilterLinear`,`kCAFilterNearest`,`kCAFilterTrilinear`)的算法就起到作用了。它作用于原图的像素上并根据需要生成新的像素显示在屏幕上。

`UIView`的`alpha`属性来确定视图的透明度。`CALayer`有一个等同的属性叫做`opacity`，这两个属性都是影响子层级的。

可以设置`CALayer`的一个叫做`shouldRasterize`属性来实现**组透明**的效果，如果它被设置为`YES`，在应用透明度之前，图层及其子图层都会被整合成一个整体的图片，这样就没有透明度混合的问题了。

`UIView`的`transform`属性是一个`CGAffineTransform`类型，用于在二维空间做旋转，缩放和平移。  
`CALayer`对应于`UIView`的`transform`属性叫做`affineTransform`。

`CGAffineTransform`中的 **“仿射” 的意思是无论变换矩阵用什么值，图层中平行的两条线在变换之后任然保持平行**。
`CGAffineTransform`是一个可以和二维空间向量(如`CGPoint`)做乘法的 3X2 的矩阵:
![Image of 3X2_Matrix](./images/3X2_Matrix.jpeg)

如下几个函数都创建了一个`CGAffineTransform`实例：
```swift
CGAffineTransformMakeRotation(CGFloat angle) // 旋转
CGAffineTransformMakeScale(CGFloat sx, CGFloat sy) // 缩放
CGAffineTransformMakeTranslation(CGFloat tx, CGFloat ty) // 平移
```

Core Graphics 还提供了一系列的函数可以在一个变换的基础上做更深层次的变换，如果做一个既要缩放又要旋转的变换，这就会非常有用了。例如下面几个函数：
```swift
CGAffineTransformRotate(CGAffineTransform t, CGFloat angle)
CGAffineTransformScale(CGAffineTransform t, CGFloat sx, CGFloat sy)
CGAffineTransformTranslate(CGAffineTransform t, CGFloat tx, CGFloat ty)
```

创建一个`CGAffineTransform`类型的空值，矩阵论中称作单位矩阵，Core Graphics 同样也提供了一个方便的常量：`CGAffineTransformIdentity`。  
如果需要混合两个已经存在的变换矩阵，就可以使用如下方法，在两个变换的基础上创建一个新的变换：
```swift
CGAffineTransformConcat(CGAffineTransform t1, CGAffineTransform t2);
```

**当按顺序做了变换，上一个变换的结果将会影响之后的变换**。

`CALayer`同样有一个`transform`属性，但它的类型是`CATransform3D`和`CGAffineTransform`类似，`CATransform3D`也是一个矩阵，但是和 2x3 的矩阵不同，`CATransform3D`是一个可以在3维空间内做变换的 4X4 的矩阵:
![Image of 4X4_Matrix](./images/4X4_Matrix.jpeg)

Core Animation提供了一系列的方法用来创建和组合CATransform3D类型的矩阵:
```swift
CATransform3DMakeRotation(CGFloat angle, CGFloat x, CGFloat y, CGFloat z)
CATransform3DMakeScale(CGFloat sx, CGFloat sy, CGFloat sz)
CATransform3DMakeTranslation(Gloat tx, CGFloat ty, CGFloat tz)
```

`CATransform3D`的透视效果通过一个矩阵中一个很简单的元素来控制：**`m34`**, 用于按比例缩放 X 和 Y 的值来计算到底要离视角多远。
`m34`的默认值是0，我们可以通过设置`m34`为`-1.0 / d`来应用透视效果，`d`代表了想象中视角相机和屏幕之间的距离，以像素为单位。

当改变一个图层的`position`，你也改变了它的灭点，做3D变换的时候要时刻记住这一点，当你视图通过调整`m34`来让它更加有3D效果，**应该首先把它放置于屏幕中央，然后通过平移来把它移动到指定位置**(而不是直接改变它的`position`)，这样所有的3D图层都共享一个灭点。

`CALayer`有一个属性叫做`sublayerTransform`。它也是`CATransform3D`类型，但和对一个图层的变换不同，它影响到所有的子图层。这意味着你可以一次性对包含这些图层的容器做变换，于是所有的子图层都自动继承了这个变换方法。

`CALayer`有一个叫做`doubleSided`的属性来控制图层的背面是否要被绘制。这是一个`BOOL`类型，默认为`YES`，如果设置为`NO`，那么当图层正面从相机视角消失的时候，它将不会被绘制。

`CAShapeLayer`是一个通过矢量图形而不是 bitmap 来绘制的图层子类。你指定诸如颜色和线宽等属性，用`CGPath`来定义想要绘制的图形，最后`CAShapeLayer`就自动渲染出来了。使用`CAShapeLayer`有以下一些优点：
- 渲染快速。`CAShapeLayer`使用了硬件加速，绘制同一图形会比用 Core Graphics 快很多。
- 高效使用内存。`CAShapeLayer`不需要像`CALayer`一样创建一个寄宿图形，所以无论有多大，都不会占用太多内存。
- 不会被图层边界剪裁掉。`CAShapeLayer`可在边界之外绘制，图层路径不会像在使用 Core Graphics 的普通`CALayer`被剪裁掉。
- 不会出现像素化。当你给`CAShapeLayer`做3D变换时，它不像一个有寄宿图的普通图层一样变得像素化。

`CATextLayer`使用了 Core text，要比`UILabel`渲染得快得多。

`CATransformLayer`不同于普通的`CALayer`，因为它不能显示它自己的内容。只有当存在了一个能作用域子图层的变换它才真正存在。`CATransformLayer`并不平面化它的子图层，所以它能够用于构造一个层级的3D结构。

`CAGradientLayer`是用来生成两种或更多颜色平滑渐变的。用 Core Graphics 复制一个`CAGradientLayer`并将内容绘制到一个普通图层的寄宿图也是有可能的，但是`CAGradientLayer`的真正好处在于绘制使用了**硬件加速**。

`CAReplicatorLayer`的目的是为了高效生成许多相似的图层。它会绘制一个或多个图层的子图层，并在每个复制体上应用不同的变换。

`CAScrollLayer`滑动的时候完全没有一个全局的可滑动区域的概念，也无法自适应它的边界原点至你指定的值。它之所以不能自适应边界大小是因为它不需要，内容完全可以超过边界。

`CATiledLayer`为载入大图造成的性能问题提供了一个解决方案：**将大图分解成小片然后将他们单独按需载入**。`CATiledLayer`的默认小图大小是 `256*256`，默认大小可以通过`tileSize`属性更改。

`CAEmitterLayer`是一个高性能的粒子引擎，被用来创建实时例子动画如：烟雾，火，雨等等这些效果。
`CAEmitterLayer`看上去像是许多`CAEmitterCell`的容器，这些`CAEmitierCell`定义了一个例子效果。一个`CAEmitterCell`类似于一个`CALayer`：它有一个`contents`属性可以定义为一个`CGImage`，另外还有一些可设置属性控制着表现和行为。

**当 iOS 要处理高性能图形绘制，必要时就是 OpenGL。应该说它应该是最后的杀手锏，至少对于非游戏的应用来说是的。**

`CAEAGLLayer`是`CALayer`的一个子类，用来显示任意的 OpenGL 图形。

**Core Animation 在每个 *run loop* 周期中自动开始一次新的*事务***。  
(run loop是iOS负责收集用户输入，处理定时器或者网络事件并且重新绘制屏幕的东西)  
即使你不显式的用`[CATransaction begin]`开始一次事务，任何在一次 run loop 循环中属性的改变都会被集中起来，然后做一次 **0.25秒** 的动画。

事务是通过`CATransaction`类来做管理，这个类不去管理一个简单的事务，而是管理了一叠不能访问的事务。

`CATransaction`没有属性或者实例方法。但是可以用`+begin`和`+commit`分别来入栈或者出栈。

任何可以做动画的图层属性都会被添加到栈顶的事务,可以通过`+setAnimationDuration:`方法设置当前事务的动画时间，或者通过`+animationDuration`方法来获取值(默认0.25秒)。

`CATranscation`接口提供的`+setCompletionBlock:`方法允许你在动画结束的时候提供一个完成的动作。

`CATransacition`有个方法叫做`+setDisableActions:`，可以用来对所有属性打开或者关闭隐式动画。

**`UIView`关联的图层禁用了隐式动画**，对这种图层做动画的唯一办法就是使用`UIView`的动画函数(而不是依赖`CATransaction`)，或者继承`UIView`，并覆盖`-actionForLayer:forKey:`方法，或者直接创建一个显式动画。

对于单独存在的图层，可以通过实现图层的`-actionForLayer:forKey:`委托方法，或者提供一个`actions`字典来控制隐式动画。

当设置`CALayer`的属性，实际上是在定义**当前事务结束之后图层如何显示的*模型***。Core Animation 扮演了一个*控制器*的角色，并且负责根据图层行为和事务设置去不断更新视图的这些属性在屏幕上的状态。

每个图层属性的显示值都被存储在一个叫做*呈现图层*的独立图层当中，可以通过`-presentationLayer`方法来访问。这个呈现图层实际上是模型图层的复制，但是它的属性值代表了在任何指定时刻当前外观效果。换句话说，可以通过呈现图层的值来获取当前屏幕上真正显示出来的值。
