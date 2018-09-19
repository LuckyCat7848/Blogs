#iOS动画(Core Animation&POP&贝塞尔&Transform)
@(学习iOS)


[toc]

-------------

##前言
动画的需求不多，用到时可能就去找 demo 代码改一改使用了。发现对很多概念都不够清晰，所以还是系统的了解一个东西更重要。在此概括整理一下，附上链接，感谢大家分享。

##一、CALayer 和 Core Animation

- [傻傻分不清：Quartz2D、QuartzCore、CoreAnimation、CoreImage、CoreGraphics](https://www.jianshu.com/p/397690fd4555)
- 细：[iOS开发系列--让你的应用“动”起来](http://www.cnblogs.com/kenshincui/p/3972100.html)
- demo：[iOS动画（Core Animation）总结](https://www.jianshu.com/p/dfe084fadfc2)
- 好书：[iOS核心动画高级技巧](https://zsisme.gitbooks.io/ios-/content/index.html)
 
###1.1 CALayer

`CALayer`是动画的载体，动画是加在 CALayer 上显示的，所以我们需要先了解一下。

>**UIView**：`UIKit`框架，用户交互使用。
**CALayer**：`QuartzCore`框架，不能响应事件，为了内容展示和动画操作。

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/CALayer.png)

`CALayer`子类：

- `CAShapeLayer`：用来根据路径绘制矢量图形。
- `CATextLayer`：绘制文字信息。
- `CATransformLayer`：使用单独的图层创建3D图形。
- `CAGradientLayer`：绘制线性渐变色。
- `CAReplicatorLayer`：高效地创建多个相似的图层并施加相似的效果或动画。
- `CAScrollLayer`：没有交互效果的滚动图层，没有滚动边界，可以任意滚动上面的图层内容。
- `CATiledLayer`：将大图裁剪成多个小图以提高内存和性能。
- `CAEmitterLayer`：各种炫酷的粒子效果。
- `CAEAGLLayer`：用来显示任意的OpenGL图形。
- `AVPlayerLayer`：用来播放视频。

`CALayer`的用处很大的，而且它并没有为所有可能的场景进行优化。为了获得`Core Animation`最好的性能，你需要为你的工作选对正确的工具。

###1.2 Core Animation

> **`Core Animation`是 iOS 和 OS X 平台上负责图形渲染与动画的基础框架。**

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/Core%20Animation.png)

**`CAAnimation`**：核心动画的基础类。不能直接使用，负责动画运行时间、速度的控制，本身实现了`CAMediaTiming`协议。

**`CAPropertyAnimation`**：属性动画的基类。通过属性进行动画设置，注意是可动画属性，不能直接使用。

- **CABasicAnimation**：基础动画。通过属性修改进行动画参数控制，只有初始状态和结束状态。

- **CAKeyframeAnimation**：关键帧动画。同样是通过属性进行动画参数控制，但是同基础动画不同的是它可以有多个状态控制。
- 
基础动画、关键帧动画都属于属性动画，就是通过修改属性值产生动画效果，开发人员只需要设置初始值和结束值，中间的过程动画（又叫“补间动画”）由系统自动计算产生。和基础动画不同的是关键帧动画可以设置多个属性值，每两个属性中间的补间动画由系统自动完成，因此从这个角度而言基础动画又可以看成是有两个关键帧的关键帧动画。

**`CAAnimationGroup`**：动画组。动画组是一种组合模式设计，可以通过动画组来进行所有动画行为的统一控制，组中所有动画效果可以并发执行。

**`CATransition`**：转场动画。就是从一个场景以动画的形式过渡到另一个场景。

##二、Facebook POP

除了 苹果系统自带的`Core Animation`动画，还有很多第三方封装动画，比较普遍的就是 Facebook 发布的`POP`动画了。POP 是2014年发布的，是个相当成熟且久经考验的框架。

- GitHub：[facebook/pop](https://github.com/facebook/pop)
- [FaceBook POP 动画详解](https://juejin.im/entry/57fb09008ac2470058be251c)
- [Facebook POP 进阶指南](http://www.cocoachina.com/ios/20140704/9034.html)
- [POP介绍与使用实践(快速上手动画)](http://adad184.com/2015/03/11/intro-to-pop/)

> **`POP`是一个来自于`Facebook`，在iOS和OS X上通用的极具扩展性的动画引擎。**

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/Facebook%20POP.png)

**`Engine`、`Utility`、`Webcore`**：微动画流畅性提供支持。
**`POPPropertyAnimation`**：属性动画。
- **POPBasicAnimation**：基础动画。
- **POPSpringAnimation**：弹性动画。
- **POPDecayAnimation**：衰减动画。

**`POPCustomAnimation`**：自定义动画。扩展性强，不止于动画功能。

> POP在基本的静态动画的基础上增加的`弹簧动画`与`衰减动画`，使之能创造出**更真实更具物理性的交互动画**。 

> 2014年POP发布后，2015年 Apple 在iOS9 系统中开始加入`CASpringAnimation`。


##三、Pop和Core Animation区别

- [iOS动画系列之---Pop和Core Animation区别](https://www.jianshu.com/p/5117eabfb8f4)
- [从CoreAnimation到Pop](https://www.jianshu.com/p/59ba5510cc26)
- [Core Animation基本概念和Additive Animation](http://studentdeng.github.io/blog/2014/06/24/core-animation/)

###3.1 Core Animation的基本原理

在Core Animation 做相关动画时，你的应用完全不会参与动画的绘制，这些动画绘制完全独立于你的应用进程。即便主线程阻塞，也不会影响你的动画执行。

> `render server`进程是真正处理动画的地方。而且线程的优先级也比我们主线程优先级高。所以有时候即使我们的 App 主线程 busy ，依然不会影响到手机屏幕的绘制工作。

###3.2 POP 动画库的基本原理

POP 本质上是基于定时器的动画库，使用每秒 60 频率的定时器，使得动画刷新绘制频率与屏幕刷新频率一致。很多这类动画库都使用 CADisplayLink 做为一个回调源。

一旦定时器刷新，动画库会计算那些活动的东西的状态（通常是layer 属性，如 bound，opactiy，transform 等）。然后动画库提供最新计算的值给有动画的 layer 由于 layer 的一些参数已经被改变，你需要通知系统在屏幕上重绘一切东西，通过这种方式来做动画的。

> POP 缺陷：
由于 POP 是基于定时器定时刷新添加动画的原理，那么如果将动画库运行在主线程上，会由于线程阻塞的问题导致动画效果出现卡顿、不流畅的情况。更为关键的是，你不能将动画效果放在子线程，因为你不能将对 view 和 layer 的操作放到主线程之外。

> POP 受主线程阻塞的影响很大，在使用过程中，应避免在有可能发生主线程阻塞的情况下使用 POP ，避免制作卡顿的动画效果，产生不好的用户体验。

###3.3 二者区别
**1. 主线程影响不同**
在主线程没有阻塞的情况下，两种动画库的表现并无差异。在主线程阻塞时，利用 POP 制作的动画视图，在每隔 1s 都会卡顿一下，而 CA 的视图却完全不受主线程阻塞的影响。

**2.  载体限制不同**
**`Core Animation`**：只能是 CALayer 。
**`Pop Animation`**：可以是任意基于 NSObject 的对象 。

**3.  使用方式不同**
**`Core Animation`**：只有 Delegate 回调方法。
**`Pop Animation`**：有 Delegate 或 Block 两种回调。

**4. 动画效果有点不一样，其在于时间函数**
**`Core Animation`**：显示动画时，默认的时间函数是 kCAMediaTimingFunctionLinear 。
**`POP`**：默认的是 kCAMediaTimingFunctionEaseOut 。只有手动把 pop 参数设置成 kCAMediaTimingFunctionLinear ，两者动画就一致了。

**5. 动画结束时不一样**
与 iOS 自带的动画不同，如果你在动画的执行过程中删除了物体的动画，那么物体会停在动画状态的最后一个瞬间，而不是闪回开始前的状态。

###3.4 CADisplayLink

- [CADisplayLink](http://www.jianshu.com/p/c35a81c3b9eb)

> **`CADisplayLink`**是一个能让我们以和屏幕刷新率相同的频率将内容画到屏幕上的定时器。我们在应用中创建一个新的 CADisplayLink 对象，把它添加到一个`runloop`中，并给它提供一个`target`和`selector`在屏幕刷新的时候调用。

**`CADisplayLink`** 与 **`NSTimer`** 有什么不同：iOS设备的屏幕刷新频率是固定的，CADisplayLink在正常情况下会在每次刷新结束都被调用，精确度相当高。

**`NSTimer`**的精确度就显得低了点，比如NSTimer的触发时间到的时候，runloop如果在阻塞状态，触发时间就会推迟到下一个runloop周期。并且 NSTimer新增了tolerance属性，让用户可以设置可以容忍的触发的时间的延迟范围。

**`CADisplayLink`**使用场合相对专一，适合做UI的不停重绘，比如自定义动画引擎或者视频播放的渲染。NSTimer的使用范围要广泛的多，各种需要单次或者循环定时处理的任务都可以使用。在UI相关的动画或者显示内容使用 CADisplayLink比起用NSTimer的好处就是我们不需要在格外关心屏幕的刷新频率了，因为它本身就是跟屏幕刷新同步的。

##四、其他

###4.1 贝塞尔曲线Bezier Path

- 公式：[贝塞尔曲线](https://blog.csdn.net/guo_hongjun1611/article/details/7842110)
- [《iOS Core Animation：Advanced Techniques》学习记录](https://www.cnblogs.com/ziyi--caolu/p/5038097.html)
- [iOS仿QQ消息动画](https://juejin.im/post/5ad71c97f265da239867cc61)
- [iOS CoreAnimation专题——技巧篇（二）CAShapeLayer with Bezier Path - Layer世界的神奇画笔](https://blog.csdn.net/u013282174/article/details/52161178)
- demo：[iOS滚珠菜单动效](http://www.code4app.com/thread-9264-1-1.html)

> 如果你想要定义曲线运动的轨迹，那么这个知识点是必须学习的（例如：QQ水滴）。

**`Bézier curve`(贝塞尔曲线)是应用于二维图形应用程序的数学曲线。** 曲线定义：起始点、终止点（也称锚点）、控制点。通过调整控制点，贝塞尔曲线的形状会发生变化。 1962年，法国数学家Pierre Bézier第一个研究了这种矢量绘制曲线的方法，并给出了详细的计算公式，因此按照这样的公式绘制出来的曲线就用他的姓氏来命名，称为贝塞尔曲线。

###4.2 CGAffineTransform与CATransform3D

- [【iOS】CGAffineTransform和CATransform3D](https://www.jianshu.com/p/f729f4a9b8e7)
- [简单代码实现TableView中Cell出现时弹出动画](http://www.code4app.com/thread-7800-1-1.html)

**`CGAffineTransform`**是用于2D层面的，操作 NSView、UIView 或者其他 2D Core Graphics 元素的.

**`CATransform3D`**是 Core Animation 的结构体，是用来做更复杂的关于 CALayer 的3D操作。CATransform3D 有着与 OpenGL 模型视图矩阵相同的内部结构，原因在于 Core Animation 是建立在 OpenGL 之上的，CALayer是 OpenGL 结构的一种封装。
