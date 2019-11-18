# iOS组件化（一）中间件设计方案### 前言

在组件化分层上，我们已经搭建了[私有库](https://www.jianshu.com/p/f4a8879693e9)，陆续添加封装好的库是一个长期项目。组件间的分离也不是一朝一夕的，所以要先把 APP 整体的组件化方案确定，让这两个长期项目同时进行。（也就是**平时多积累私有库，和利用中间件逐渐使工程模块化。**）

首先学习下大神的思想，下面两个是最有名的方案了：

### 方案一：MGJRouter

[MGJRouter](https://github.com/meili/MGJRouter)的思想是使用的服务在 `+(void)load` 中都先去注册 URL ，用到的时候根据 URL 在已注册的字典中查找对应的实现。

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/MGJRouter.png)

缺点：
1. URL 是注册时写死的；
2. 需要在`+(void)load`方法提前注册，写法不友好；
3. MGJRouter 使用方式有两种，一个是 URL 跳转，一个是协议的方法调用。如图所示：协议的方法调用中，公共的协议注册类和每个使用的地方会耦合，而且因为是方法调用，所以协议类也会暴露。使用如下：

```
// 协议管理类EHIServiceProtocolManager暴露
id<EHISelfDrivingServiceProtocol> selfService = [EHIServiceProtocolManager serviceProvideForProtocol:@protocol(EHISelfDrivingServiceProtocol)];

// 因为方法调用，要获取到协议来调用，对应的协议EHISelfDrivingServiceProtocol也暴露
EHISelfStep1View *view = [selfService step1ViewWithCityName:@"大连"];
```

### 方案二：CTMediator

[CTMediator](https://github.com/casatwy/CTMediator)的思想是 Target-Action。不需要注册，通过 Target 类名和 Action 方法名，利用 Runtime 找到对应的实现。

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/CTMediator.png)

缺点：
1. 因为利用 Runtime 反射，本地 Target 和 Action 的命名规则必须规范，URL 也按照该规范来。

### 最终方案：LCMediator

两个方案都写了 demo 来熟悉，最后大思想上选择了 casa 大神的中间件方案，细节部分还是有学习 MJRouter 。

因为大神毕竟是大神，写这么长的[博客](https://casatwy.com/iOS-Modulization.html)来教我们思想，然后 demo 写的就不是很完善。例如：没有 MGJRouter 的回调，没有异步处理。

所以学习过程中接合两种方案，自己写了一个 [**LCMediator**](https://github.com/LuckyCat7848/LCMediator) 。

#### 1. 整体介绍

```
- LCMediator
	- LCMediator                 // 🔥中间件主要实现
	- LCMediatorService          // 🔥拦截URL判断、统一处理错误
	- LCNoServiceViewController  // 无服务页面,可根据自己的项目自行替换
- Example
	- SelfDriving
		- LCSelfDrivingService   // 🔥各模块提供的服务类
		- LCMediator+SelfDriving // 也可以使用分类的方式调用
	- Chauffeur
		- LCChauffeurService
	- User
		- LCUserService
	- Account     
		- LCAccountService
```

#### 2. 主要实现

```
/**
 *  远程App调用入口
 
 *  urlString  scheme://[target]/[action]?[params]
 *  example    lc://targetA/actionB?id=1234
 */
+ (void)openURL:(NSString *)urlString;

/** 远程App调用入口（有回调） */
+ (void)openURL:(NSString *)urlString
     completion:(void(^)(id result, NSError *error))completion;

/** 本地组件调用入口 */
+ (void)openTarget:(NSString *)targetName
            action:(NSString *)actionName
            params:(NSDictionary *)params;

/** 本地组件调用入口（有回调） */
+ (void)openTarget:(NSString *)targetName
            action:(NSString *)actionName
            params:(NSDictionary *)params
        completion:(void(^)(id result, NSError *error))completion;

/** 释放target（参数为模块名,如：user/account等） */
+ (void)releaseCachedTarget:(NSString *)targetName;
```

#### 3. 使用方式

使用方式参考 demo，主要关注：
1. 有无参数：
2. 有无回调需求（已处理除了异步自己在需要的地方回调`kLCMediatorNoResponseCompletion()`外，其他都默认回调）；
3. 异步处理。

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/LCMediator_Demo.png)

#### 4. 在现有工程中实施基于LCMediator的组件化方案

1. 每个模块提供自己的 Service；
2. 需要的话改前缀，使用自己公司的前缀；
3. 逐渐完善 Service 提供的方法。

### 结尾

这个中间件写了很久才放在项目中使用，看了很多文章，还有的朋友帮忙提供思路，有的朋友帮忙 review。也在项目中使用了段时间，这才拿出来，还有问题的话友友们尽管提出，谢谢~
