# iOS项目重构总结(AppDelegate、基类、Const)
@(学习iOS)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [iOS项目重构总结(AppDelegate、基类、Const)](#ios%E9%A1%B9%E7%9B%AE%E9%87%8D%E6%9E%84%E6%80%BB%E7%BB%93appdelegate%E5%9F%BA%E7%B1%BBconst)
- [一、背景](#%E4%B8%80%E8%83%8C%E6%99%AF)
- [二、重构的方法](#%E4%BA%8C%E9%87%8D%E6%9E%84%E7%9A%84%E6%96%B9%E6%B3%95)
- [2.1 整理所有方法并划分功能块](#21-%E6%95%B4%E7%90%86%E6%89%80%E6%9C%89%E6%96%B9%E6%B3%95%E5%B9%B6%E5%88%92%E5%88%86%E5%8A%9F%E8%83%BD%E5%9D%97)
- [2.2 思考功能块去处](#22-%E6%80%9D%E8%80%83%E5%8A%9F%E8%83%BD%E5%9D%97%E5%8E%BB%E5%A4%84)
- [2.3 思考功能块用法](#23-%E6%80%9D%E8%80%83%E5%8A%9F%E8%83%BD%E5%9D%97%E7%94%A8%E6%B3%95)
- [三、AppDelegate](#%E4%B8%89appdelegate)
- [四、Template](#%E5%9B%9Btemplate)
- [五、Const](#%E4%BA%94const)
- [六、总结](#%E5%85%AD%E6%80%BB%E7%BB%93)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

----------

## 一、背景

公司的项目到现在已有 8 年的开发时间了，相较上技术发展之快，里面有很多不合理的地方也不足为奇。开发起来困难且影响速度，所以我们在一步步的改进着。

**前期工作：**

六月份的时候，我们对项目进行了`项目目录`上的整理。按照模块、功能划分所有文件，删除了很多不再使用的文件，整理了很多虚拟目录和实际文件位置问题。对后面的开发使用效果出乎意料的重要，真正体会到一个好的项目目录结构是一个`App`重要的开端。这半年来的开发过程中，也在`不断完善`目录的设计。具体可以参考[Objective-C开发规范(包括项目目录结构)](https://www.jianshu.com/p/e12560a0ead2)。

提到这呢，一是想强调项目目录的重要性，二是因为这次的重构也涉及到了目录的改动，更新了之前的文档，希望大家了解到。

**本次工作：**

尝到了好的目录的甜头，继续我们的优化之路。这次呢，我们将改进重心放到了重要模块的重构。

**AppDelegate 是每个iOS项目的核心**，是整个 App 的入口，它确保应用程序和系统以及和其他应用程序之间的交互，所以 AppDelegate 通常会承担很多职责。而因为它的唯一性和一直存在的特征，很多方法和功能放在这里特别方便使用，这更加重了 AppDelegate 的负担，最终导致了`Massive AppDelegate`。

**Template 是每个页面都要继承的父类**，除了统一的页面配置，还有为了方便某些页面使用的同个功能都放在了这里。在修改某个功能时，一点改动都牵扯所有页面；而且继承的缺点使得开发者必须要了解父类，庞大的父类无疑增加开发的成本，之前就有过某个页面使用了和父类一样的属性而导致的 Bug。

**Const是全局使用的配置文件**，之前架构的不合理导致为了方便使用的定义就都放在 Const 中，导致 Const 庞大且混乱。

---------

综上所述，在复杂的 AppDelegate 和 Template 里修改任何东西的成本都是很高的，因为它将会`影响整个APP`，且包括 Const 的全局性，都增加了文件编译时的复杂度和性能问题。毫无疑问，`这三个模块的简洁和清晰对一个好的 iOS 架构来说至关重要`。

因为这它们的改动将会影响整个 App 的，所以既然是必须要做的改动，我们就用一个版本的开发时间将这三个模块一起改动，这样也少给测试同学增加麻烦嘛~✌️

## 二、重构的方法

### 2.1 整理所有方法并划分功能块

`AppDelegate`、`Template`、`Const`因为方便使用，所以里面被放很多不属于它们的功能。

例如：
1. AppDelegate，包含配置`SDK`、网络异常弹框、用户自动登录等；
2. Template，包含部分页面都用到的属性名、支付功能、地图定位功能等；
3. Const，包含有各个模块的枚举、埋点名称、通知名称等。

所以整理某一块的时候，首先整理里面的所有方法，然后按照功能划分几个功能块。

### 2.2 思考功能块去处

例如，**AppDelegate**中：
1. 十几个`SDK`的配置方法，应该单独有一个`SDK`配置类去统一管理；（配置）
2. 关于网络的弹窗，应该拿到弹窗类中管理调用；（工具）
3. 用户自动登录，应该在用户信息管理中。（业务）

这些配置、工具类方法、业务功能，还有很多属性、方法都不属于 AppDelegate，是为了使用方便而放在这的。

### 2.3 思考功能块用法

功能块的去处要么是在放在它本该去的类中，要么是单独创建一个类。单独创建的类就要考虑是用`单例`还是`类方法`，或是使用`分类`。

例如，**AppDelegate**中：

1. 需要`App`启动后立马调用的、功能间没有太大关系的一系列方法，使用类方法即可；
2. 而地图定位功能就需要使用单例了；
3. `AppDelegate`本身方法的实现使用分类。

## 三、AppDelegate

重构前：**`.h`：126 行；`.m`：1909 行**。  
重构后：**`.h`：16行；`.m`：84 行**。

重构后的`AppDelegate.h`：

```
#import <UIKit/UIKit.h>

@interface AppDelegate : UIResponder <UIApplicationDelegate>

@property (nonatomic, strong) UIWindow *window;

@end
```

重构后的`AppDelegate.m`：

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 配置APP运行环境
    // 配置SDK
    // 业务相关(需要启动后马上要做的操作)
    return YES;
}
```

最终主要目录结构如下：

```
|- AppDelegate      // 单独一个文件夹   
    |- AppDelegate  // 入口.h/.m文件
    |- Category     // AppDelegate的扩展
        |- AppDelegate+EHI3DTouch
        |- AppDelegate+EHIOpenURL
        |- AppDelegate+EHIRemoteNotification
    |- StartLanch   // App启动后需要的业务逻辑
        |- EHIStartLanchManager
    |- SDKManager   
        |- EHIVendorSDKDefines.h // SDK的key或ID等定义
        |- EHIVendorSDKManager   // 第三方SDK配置管理
        |- EHIMapManager         // 地图管理
        |- EHIPushManager        // 推送管理
```

最终`AppDelegate.h`保留了项目初建时的样子，不提供外部多余的属性和方法。

1. 本身功能的扩展；
2. `App`启动后需要调用的业务逻辑；
3. `SDK`配置管理，重要`SDK`的封装使用。
4. 其他较为散的功能放在对应类中。例如：城市信息放在城市数据管理类中；
5. 模块功能封装后放在对应的模块文件夹下。例如：下单流程数据操作单独封装，放在`Data`模块的文件夹下。

> 总之，主要本着**单一职责原则**，按职责划分来处理。

## 四、Template

重构前：`.h`：292 行；`.m`：1256 行。  
重构后：`.h`：21行；`.m`：53 行。

重构后的`Template.h`：
不提供任何属性和方法，只引入了必须的头文件。如：网络请求的头文件。

重构后的`Template.m`：
只实现了系统本身的生命周期方法。处理：导航栏设置、页面统计。

最终主要目录结构如下：

```
|- Base // 单独一个文件夹  
    |- Template      // .h/.m文件
    |- NavigationBar // 导航栏处理
        |- UIViewController+EHINavigationBar
```

功能整理如下：
1. 老逻辑方法的删除。例如删除旧版的微信支付方法、相关文件，连带`Html5`页面回调的处理确定不会使用后，删除相关方法；
2. 支付、地图、定位、登入登出模块功能分别封装；
3. 为了方便写的属性，分别写在各个需要的`ViewController`中；
4. 为了方便写的方法，找到各自去处。例如：字符串判断放在`NSString`的扩展类中；
5. 导航栏设置单独在分类中实现；
6. 作为基类，导入头文件的控制。

## 五、Const

重构前：`.h`：1035 行；`.m`：14 行。  
重构后：`.h`：56行。

最终主要目录结构如下：

```
|- Defines                // 整个应用会用到的定义
    |- EHIMacro.h         // 必用头文件引入（YYKit.h）
    |- EHIAppDefines.h    // App信息、UI的宏定义(字体、颜色、适配)
    |- EHIFunctionDefines // 全局方法或内敛函数定义（EHIWeakSelf/EHIStrongSelf）
    |- EHIEventDefines    // 统计事件使用的Key
```

从目录中可以看出来：

1. 主要是统计事件使用的`Key`定义，`App`信息、`UI`的定义，全局方法的定义三大块；
2. `EHIMacro.h`作为宏控制在`.pch`中全局引入的头文件，只引入一些必须的，不必须的各个文件用的时候再引用；
3. 其他的枚举、宏定义、常量值等都在对应的模块中分别创建`Defines`文件处理。例如：订单中心的`EHIOrderCenterDefines`。

这部分的整理中，因为对只创建`.h`文件和也创建`.m`文件引起了争议，所以扩展了[iOS常量使用extern和static原理探究，为什么Defines中默认推荐使用extern?](https://www.jianshu.com/p/00dc973db590)这篇博客，有兴趣的可以看一下。

## 六、总结

1. 不要为了方便把属性、方法、功能逻辑放在`AppDelegate`和`Template`中；
2. 不要为了方便部分类把定义的间距或尺寸放在`Const`中，要写模块`Defines`或使用`extern`使多个文件共用；
3. 加全局使用的东西之前检查是否已经有可用的；
4. 不用的东西要删除：包括类文件、方法、属性、引入的头文件；
5. 少用单例；
6. 少用`BOOL`属性，同级判断用枚举；
7. 再次强调**单一职责原则**。

------

嗯。。。这次任务很多很辛苦，但是学习到了很多，开心。✌️

