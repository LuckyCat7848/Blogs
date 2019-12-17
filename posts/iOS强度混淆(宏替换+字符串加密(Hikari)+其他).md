# iOS强度混淆(宏替换+字符串加密(Hikari)+其他)

## 前言

网上有的混淆是为了安全或做马甲包，是可以上 App Store 的轻度混淆。本篇文章说的是公司需要通过一些渠道的过审，而做的强度混淆，没试过上线。

（PS：由于项目老旧庞大，本篇文章记录遇到问题及解决方式，应该是很详细的了😭。）

----------

强度混淆一般重要的是代码混淆、字符串加密、其他（例如：防越狱、防截屏、防录屏等）。

## 一、逆向工具

逆向的方式有很多，最基础的就是 class-dump（可以查看所有头文件），深度的可以用 Hopper。

参考：[# iOS进阶 # 逆向的了解](https://www.jianshu.com/p/f74f3aa1cc68)

可以在开始之前逆向一下看看，混淆后对比一下。

## 二、代码混淆（宏替换）

### 2.1 宏替换原理

宏替换指的是`类名`、`方法名`、`属性名`的混淆，即将这些生成混淆宏，在预编译时期进行替换。

### 2.2 STCObfuscator

> [STCObfuscator](https://github.com/chenxiancai/STCObfuscator) 是用来进行 Object-C 代码混淆的工具，在模拟器 DEBUG 环境下运行生成混淆宏，混淆的宏可以在其他环境下进行编译，支持 Cocoapod 代码混淆。

使用方式很简单，作者步骤很详细。但是我的项目比较复杂，在使用中遇到些问题，分享一下。

### 2.3 筛选需要混淆的类名

```
/** 不进行混淆的类 */
@property (nonatomic, strong) NSArray *unConfuseClassNames;

/** 不进行混淆的带特性前缀的类 */
@property (nonatomic, strong) NSArray *unConfuseClassPrefix;

/** 不进行混淆的带有特定前缀的符号名 */
@property (nonatomic, strong) NSArray *unConfuseMethodPrefix;
```

1. 如上，首先工具提供了不进行混淆的类或前缀，看起来很简单，把所有第三方的前缀加进去就可以了。但是我们的项目很大很旧，找出所有第三方前缀，实验了几次还是很多错误，尤其是有些没有前缀或 `.a` 根本看不到的一些，总之是错误很多。

2. 换个思路，我们来添加需要混淆的类，当然需要改源码，改动不大，可以接受。如果你的项目结构和代码足够好，恭喜你写上你们的统一前缀就行了。但是如果你和我一样遇到项目很旧，没有统一的前缀就需要`筛选出所有要进行混淆的类名`。

我们采用的方式：
1. 我们的项目结构目录结构是整理过的，所以明确的知道哪些是项目代码（项目目录整理见之前文章）；
2. 使用脚本把所有要混淆的类名输出在`.txt`文档中（待补充）。

----------
简单脚本[教程](https://www.runoob.com/w3cnote/shell-quick-start.html)：

1. 新建文件夹 `getClassName`；
2. 在改文件夹下新建 `job.sh `脚本文件；
3. 脚本文件内容为 `#!/bin/bash`;
4. 依次执行下面的命令：
```
➜  ~ /Users/zhangmaomao/Desktop/getClassName
➜  getClassName chmod +x job.sh
➜  getClassName ./job.sh
➜  getClassName find ./Classes -name "*.h">path.txt
```
5. `path.txt`中得到所有`.h`文件的路径名如下：
```
./Classes/Home/EHIHomeDefines.h
./Classes/Home/Activity/Models/FocusModel.h
./Classes/Home/Activity/ViewControllers/HomeActivityViewController.h
./Classes/Home/Activity/Views/HomeActivityCell.h
...
```
6. 删掉类名前面路径：
```
➜  getClassName book=/Users/zhangmaomao/Desktop/getClassName/path.txt
➜  getClassName while read line
while> do
while> echo ${line##*/} >>result.txt
while> done <$book
```

### 2.4 筛选不混淆的model类名

经过上面的过程，混淆后终于`运行起来`了！（注意该工具是 Debug 下生成 #define 混淆宏，然后使用宏替换。仔细看下代码，直接运行的时候注释相关代码）

但是，`数据不正常显示`。排查了以后发现又是老项目的坑😭。排查原因：model 类接收数据没有一次性解析，第二层数据获取问题。因为 model 的属性已经混淆了，再去去数据自然是没有的，代码例如：

```
self.storeStockResponse = [StoreStockResponse objectWithKeyValues:chauffeurResult.Result];
self.filterTitleList = [[CarFilterType objectArrayWithKeyValuesArray:self.storeStockResponse.CarFilterTypeList] mutableCopy];
```

现在就需要筛选出不混淆的 model 类名，嗯。。如果你有基类或统一前缀的话就 happy 了，不幸的是我没有。。。需要的话操作如下：
1. 脚本找出调用数据解析的类名，这是必须要筛掉的（待补充）；
2. 除了获取到的数据解析，还会有 model 类转 json 数据请求接口的情况（代码例如下），对象调用的就没有办法像上面那样拿到调用者，所以保险起见我们的情况需要筛选出所有 model 类；
```
[contactModel keyValues]
```
3. 不幸的是我们也没有统一的后缀。。。只能先筛选多数的 model、Response、Request、Dto 结尾的类，还有熟悉重要的挑几个；
4. 结合上面脚本筛选出来的，就是一堆不需要筛选的类了。

### 2.5 特殊字符

还有一点是我筛选掉了`failHandler`，因为很多第三方里包含在此，混淆就拿不到回调了。

```
[STCObfuscator obfuscatorManager].unConfuseMethodPrefix = @[@"failHandler"];
```

### 2.6 开始混淆

从原来要筛选的文档中删除掉不混淆的 model 类名，清空`STCDefination.h`文件，重新在 Debug 环境下在模拟器上运行。

使用宏替换运行，终于，`程序运行正常`！

（运行时看不出来混淆效果，这时候 class-dump 出来的头文件已经明显混淆过了）

## 三、字符串加密（Hikari）

直到这里是还是不够的，项目中难免有明文字符串，比如加密钥字符串等，敏感字符的混淆是必须的！！！

所以又使用了这个工具：[Hikari](https://www.jianshu.com/p/bfba47ec9695)。

这个工具很强大，有以下几种配置可以选择：

```
-mllvm -enable-bcfobf 启用伪控制流
-mllvm -enable-cffobf 启用控制流平坦化
-mllvm -enable-splitobf 启用基本块分割
-mllvm -enable-subobf 启用指令替换
-mllvm -enable-acdobf 启用反class-dump
-mllvm -enable-indibran 启用基于寄存器的相对跳转，配合其他加固可以彻底破坏IDA/Hopper的伪代码(俗称F5)
-mllvm -enable-strcry 启用字符串加密
-mllvm -enable-funcwra 启用函数封装
-mllvm -enable-allobf  依次性启用上述所有标记
```

`-mllvm -enable-strcry`是必须的，问题也不大的。如果你的项目很大，建议不要贪多，加上以后打包会慢，`打出来的包先安装后看一下是不是可以的<否则就白话时间试这么配置了>`，你的项目是否可以支持这么多的选项配置。如果不行的话需要再排查下具体原因了。

## 四、其他

其他的就很简单了，看你要过的审核具体要求了，大概说几个重要的。

### 4.1 防越狱、防截屏


`SavetyDetectionTool.h`：
```
#import <Foundation/Foundation.h>
@interface SavetyDetectionTool : NSObject

+ (instancetype)sharedSavetyDetection;

- (void)detect;
@end
```

`SavetyDetectionTool.m`：
```
#import "SavetyDetectionTool.h"
#import <UMMobClick/MobClick.h>
#import <ReplayKit/ReplayKit.h>

static SavetyDetectionTool *savetyDetect = nil;
@implementation SavetyDetectionTool

+ (instancetype)sharedSavetyDetection {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        savetyDetect = [SavetyDetectionTool new];
    });
    return savetyDetect;
}

- (void)detect {
    if ([MobClick isJailbroken]) {
        UIAlertView *aleter = [[UIAlertView alloc] initWithTitle:@"提示" message:@"您的设配已越狱，请注意保护隐私安全" delegate:nil cancelButtonTitle:nil otherButtonTitles:@"知道了", nil];
        [aleter show];
    }
  
    //注册通知
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(userDidTakeScreenshot)
                                                 name:UIApplicationUserDidTakeScreenshotNotification object:nil]; 
    
}
- (void)userDidTakeScreenshot {
    
    UIAlertView *aleter = [[UIAlertView alloc] initWithTitle:@"提示" message:@"监测到设备正在截屏，请注意安全防护" delegate:nil cancelButtonTitle:nil otherButtonTitles:@"知道了", nil];
    [aleter show];
}
@end
```

使用，`AppDelegate.m`：
```
[[SavetyDetectionTool sharedSavetyDetection] detect];
```

### 4.2 防录屏

基类`UIViewController.m`：

```
- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:YES];

    // 监测当前设备是否处于录屏状态
    UIScreen *screen = [UIScreen mainScreen];
    if (@available(iOS 11.0, *)) {
        if (screen.isCaptured) {
            [self screenshots];
        }
        [[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(screenshots) name:UIScreenCapturedDidChangeNotification object:nil];
    }
}

- (void)screenshots {
    UIAlertView *aleter = [[UIAlertView alloc] initWithTitle:@"提示" message:@"监测到设备正在录屏，请注意安全防护" delegate:nil cancelButtonTitle:nil otherButtonTitles:@"知道了", nil];
    [aleter show];
}

- (void)dealloc {
    if (@available(iOS 11.0, *)) {
        [[NSNotificationCenter defaultCenter] removeObserver:self name:UIScreenCapturedDidChangeNotification object:nil];
    }
}
```

### 4.3 只保留验证码登录方式

看审核需求。

## 五、总结

最后有大佬帮忙逆向了下，这次的混淆还是不错的，但愿顺利过审。后面可以再研究下更好的加密方式，但是，更重要的是我们必须优化好我们的项目！

根据这次的混淆，大概总结重点注意一下几点：

#### 5.1 统一前缀

组件库：`SEED`；

自驾：`EHI`；
专车：`EHIC`；
国际租车：`EHII`；
出租车：`EHIT`;

之后改动统一修改。

#### 5.2 统一类名规范

UIViewController：以`ViewController`结尾；
ViewModel：以`ViewModel`结尾（ViewModel 的扩展类名称不用再加`EHI`）；
UIView：以`View`结尾；
Model：以`Model`结尾；

工具类扩展，除大功能外，主功能用`+EHICategory`格式。例如：
```
NSString+EHICategory
NSString+EHIAES
```

#### 5.3 统一数据解析方式

统一使用`EHIModel.h`。

#### 5.4 统一图片加载方式

统一使用`EHIWebImage.h`。

#### 5.5 统一目录结构、第三方的使用

无论是自己写的，还是第三方放进去的，明确文件的位置，具体参考之前的目录规范。
