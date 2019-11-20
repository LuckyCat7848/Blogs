# iOS组件化（二）模块生命周期独立化

虽然我们重构了`AppDelegate`，给`AppDelegate`瘦身，但是还是会有各模块耦合在一起的情况。所以，组件化必须要考虑到各模块的生命周期独立化，也就是`AppDelegate`中的生命周期方法要让**每个模块都单独管理一份**。

## 豆瓣FRDModuleManager

- [豆瓣模块管理FRDModuleManager](https://github.com/lincode/FRDModuleManager)

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/FRDModuleManager.png)

> 豆瓣的方案是使用一个`.plist`文件配置要使用 APP 生命周期的类，再使用一个`FRDModuleManager`负责调用这些类。

FRDModuleManager 是遵循`<UIApplicationDelegate, UNUserNotificationCenterDelegate>`协议的。如下：

```
interface FRDModuleManager : NSObject<UIApplicationDelegate, UNUserNotificationCenterDelegate>

+ (instancetype)sharedInstance;

- (void)loadModulesWithPlistFile:(NSString *)plistFile;

- (NSArray<id<FRDModule>> *)allModules;

end
```

使用：在 `AppDelegate`中使用`FRDModuleManager`加载`.plist`文件中的类，并在 AppDelegate 的生命周期方法中调用 FRDModuleManager 的方法。示例如下：

```
- (BOOL)application:(UIApplication *)application willFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

  NSString* plistPath = [[NSBundle mainBundle] pathForResource:@"ModulesRegister" ofType:@"plist"];

  FRDModuleManager *manager = [FRDModuleManager sharedInstance];
  [manager loadModulesWithPlistFile:plistPath];

  [manager application:application willFinishLaunchingWithOptions:launchOptions];

  return YES;
}
...
```

缺点：需要引入`FRDModuleManager.h`，而且每个方法都要调用`FRDModuleManager`。

实际应用中，每个已开发的工程都是有`AppDelegate.m`文件的，而且如果 AppDelegate 使用了扩展，还需要在扩展中调用，FRDModuleManager 的使用就散落了，也容易遗漏。

## 支付宝mPaaS

- [支付宝mPaaS](https://juejin.im/post/5bdc19cbf265da614b117217)

> 通过修改 main.m 函数的实现，mPaaS 框架使用自己的 ClientDelegate 类接管了 UIApplicationDelegate 中各种 App 生命周期。mPaaS 框架接入之后，ClientDelegate 完全替代了一般工程中的 AppDelegate 的角色，从而实现了整个应用的生命周期都是由框架进行管理。

```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        // Now use mPaaS framework
        return UIApplicationMain(argc, argv, @"Application", @"ClientDelegate"); 
    }
}
```

## 我们

- [项目地址~](https://github.com/LuckyCat7848/LCMediator)

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/APPDelegateClient.png)

图中，在`.plist`中配置了每个模块独立管理 App 生命周期的类名。

然后，学习了`mPaaS`的思路，在`main.m`中改变 App 启动接收者为`LCClientDelegate`：

```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, @"LCClientDelegate");
    }
}
```

`LCClientDelegate`同样需要遵循`<UIApplicationDelegate, UNUserNotificationCenterDelegate>`协议，接管了 App 声明周期以后，把接收到的 App 生命周期方法利用**消息转发**给`.plist`中配置的所有接受者。

优点：不需要改变原来的`AppDelegate.m`文件，将它也配置在`.plist`文件中即可。

## 但是

最终的方案的确很好，毕竟`mPaaS`可是收费的，借鉴它的思想自然不错。

兴冲冲的学习了以后，加入到项目中没多久，就发现一般项目中用不到这么大的需求😭。主要还是因为 AppDelegate 本身就不需要很大的逻辑，我们之前的逻辑很多是不对的，一个好的项目结构不需要如此。例如：通知需求，也可以轻松使用中间件化解耦合。

它适合的场景就像`mPaaS`中提到的**`微应用`**概念，例如支付宝和微信的小程序。而我们只不过是一个项目的不同模块而已。

那你会想如果使用的话最多不过是大材小用？因为利用的是`NSProxy消息转发`的方案，常使用 APPDelegate 的 window 属性都要走在里面不断的转发，就很烦了。。。

建议：最终建议如果类似支付宝和微信的的小程序概念，也就是当前只运行一个或几个程序（不是全部），在原来的`.plist`注册上优化只转发给正在使用的微应用。
