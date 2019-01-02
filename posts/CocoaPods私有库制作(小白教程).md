# CocoaPods私有库制作(小白教程)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

  - [一、背景](#%E4%B8%80%E8%83%8C%E6%99%AF)
  - [二、准备私有库平台](#%E4%BA%8C%E5%87%86%E5%A4%87%E7%A7%81%E6%9C%89%E5%BA%93%E5%B9%B3%E5%8F%B0)
  - [三、创建一个Git远程仓库](#%E4%B8%89%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AAgit%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93)
  - [四、创建Pod代码库](#%E5%9B%9B%E5%88%9B%E5%BB%BApod%E4%BB%A3%E7%A0%81%E5%BA%93)
    - [4.1 使用`pod`创建库：](#41-%E4%BD%BF%E7%94%A8pod%E5%88%9B%E5%BB%BA%E5%BA%93)
    - [4.2 添加文件，写好Demo](#42-%E6%B7%BB%E5%8A%A0%E6%96%87%E4%BB%B6%E5%86%99%E5%A5%BDdemo)
    - [4.3 编辑`*.podspec`](#43-%E7%BC%96%E8%BE%91podspec)
    - [4.4 验证`*.podspec`](#44-%E9%AA%8C%E8%AF%81podspec)
    - [4.5 提交代码](#45-%E6%8F%90%E4%BA%A4%E4%BB%A3%E7%A0%81)
    - [4.6 推送`*.podspec`到远程](#46-%E6%8E%A8%E9%80%81podspec%E5%88%B0%E8%BF%9C%E7%A8%8B)
    - [4.7 验证远程是否通过](#47-%E9%AA%8C%E8%AF%81%E8%BF%9C%E7%A8%8B%E6%98%AF%E5%90%A6%E9%80%9A%E8%BF%87)
    - [4.8 pod搜索](#48-pod%E6%90%9C%E7%B4%A2)
  - [五、使用](#%E4%BA%94%E4%BD%BF%E7%94%A8)
  - [六、参考](#%E5%85%AD%E5%8F%82%E8%80%83)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

------

## 一、背景

首先了解下使用需求。公司有多个项目同时进行，期望可以共用一个工具库。而该工具库代码可能涉及到公司内部信息，不便公开，需要部署在内部服务器上。也就是私有库的管理。

大家都知道`CocoaPods`管理第三方库十分方便，所以我们希望私有库的使用也可以借助`CocoaPods`管理。


## 二、准备私有库平台

首先，我们需要一个管理私有库代码的平台，我们公司暂时使用的是`GitLab`，先了解下概念。

`Git`：版本控制系统。

`Github`：在线的基于 Git 的代码托管服务，同时提供付费账户和免费账户。这两种账户都可以创建公开的代码仓库，只有付费账户可以创建私有的代码仓库。

**`Gitlab`**解决了这个问题, 可以在上面**创建免费的私人`repo`**。

如何搭建本地服务器`Gitlab`仓库管理，大家自己去看吧。

## 三、创建一个Git远程仓库

> 我们私有库包含多个工具类的代码库，每个工具库代码使用`CocoaPods`打包后会生成一个`.podspec`文件，来描述该工具库的具体信息，包括代码地址。而所有的库的`.podspec`文件要有一个`Spec Repo`**私有仓库**去管理。

多个**`.podspec`**文件也称`specs`，作为我们查找库时候的一个索引，为什么我们执行`pod search AFNetworking`命令时，返回结果如此之快。因为安装`CocoaPods`的时候，本地目录就已经有了一份`master`（公开）的`specs`，全球程序员们提交到`CocoaPods`的开源代码在这都有记录。

我们可以通过执行命令查看一下目录结构，结果一目了然。命令如下：

```
open ~/.cocoapods/repos/master
```

我们创建的私有`specs`仓库地址为`http://私有库地址/EHILibraryiOS/EHILibrarySpecs.git`。下面执行命令把`Spec`创建到本地。命令如下：

```
pod repo add EHILibrarySpecs http://私有库地址/EHILibraryiOS/EHILibrarySpecs.git
```

这时候`EHILibrarySpecs`就在本地目录下创建成功了，通过命令查看`EHILibrarySpecs`的目录结构，会发现里面的内容和`git`仓库上的保持一致。`EHILibrarySpecs`和`master`在目录中同级。

```
open ~/.cocoapods/repos/EHILibrarySpecs
```

## 四、创建Pod代码库

有了仓库，我们就可以添加代码库啦。

### 4.1 使用`pod`创建库：

```
pod lib create EHIHiCarBluetooth
```

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/CocoaPods_cmd.png)

过程中终端会向我们提出一系列问题，大家自行选择即可。

### 4.2 添加文件，写好Demo

如图，添加文件：

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/CocoaPods_category.png)

工程目录如下：

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/CocoaPods_project.png)

在`Example`中写好`Demo`，保证编译运行ok。

### 4.3 编辑`*.podspec`

> 务必保证填写的每一项都要是正确的，比如主页和链接都要能够访问。

编辑`EHIHiCarBluetooth.podspec`文件如下：

```
#
# Be sure to run `pod lib lint EHIHiCarBluetooth.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see https://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
    s.name             = 'EHIHiCarBluetooth'
    s.version          = '0.1.0'
    s.summary          = '一句话描述'
    
    # This description is used to generate tags and improve search results.
    #   * Think: What does it do? Why did you write it? What is the focus?
    #   * Try to keep it short, snappy and to the point.
    #   * Write the description between the DESC delimiters below.
    #   * Finally, don't worry about the indent, CocoaPods strips it!
    
    s.description      = <<-DESC
    TODO: Add long description of the pod here.
    DESC
    
    s.homepage         = 'http://私有库地址/EHILibraryiOS/EHIHiCarBluetooth'
    # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
    s.license          = { :type => 'MIT', :file => 'LICENSE' }
    s.author           = { '用户名' => '邮箱地址' }
    s.source           = { :git => 'http://私有库地址/EHILibraryiOS/EHIHiCarBluetooth.git', :tag => s.version.to_s }
    # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'
    
    s.ios.deployment_target = '8.0'
    
    s.source_files = 'EHIHiCarBluetooth/Classes/*'
    
    # s.resource_bundles = {
    #   'EHIHiCarBluetooth' => ['EHIHiCarBluetooth/Assets/*.png']
    # }
    
    # s.public_header_files = 'Pod/Classes/**/*.h'
    # s.frameworks = 'UIKit', 'MapKit'
    
    s.dependency 'YYKit'
    s.dependency 'BabyBluetooth', '~> 0.7.0'
end
```

### 4.4 验证`*.podspec`

验证本地库是否通过验证，终端输入如下命令：

```
pod lib lint --use-libraries --allow-warnings
```

`--use-libraries`表示使用了第三方，`--allow-warnings`表示忽略警告。

如果使用了私有库，命令如下（我的这个没有用到）：

```
pod lib lint --sources=http://私有库地址/EHILibraryiOS/EHIHiCarBluetooth.git,https://github.com/CocoaPods/Specs.git --use-libraries --allow-warnings
```

### 4.5 提交代码

```
git add .

git commit -m 'create EHIHiCarBluetooth 0.1.0'
```

如果你还未创建远程仓库，你需要创建与之对应的远程仓库：

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/CocoaPods_gitlab.png)

创建之后与本地仓库关联并推送，在终端执行如下命令：

``` 
git push --set-upstream http://私有库地址/EHILibraryiOS/EHIHiCarBluetooth.git master
```

提交完成之后进行打标签操作（这就作为该库的版本，版本号和上面`.podspec`文件配置中保持一致）：

```
git tag -a 0.1.0 -m 'release version 0.1.0'

git push http://私有库地址/EHILibraryiOS/EHIHiCarBluetooth.git 0.1.0
```

### 4.6 推送`*.podspec`到远程

 首先将本地`EHIHiCarBluetooth.podspec`推送到远程私有`EHILibrarySpecs`仓库：

```
pod repo push EHILibrarySpecs EHIHiCarBluetooth.podspec --use-libraries
```

`--force`一般用不着，需要强制推送的话可以在后面加上。

### 4.7 验证远程是否通过

```
pod spec lint EHIHiCarBluetooth.podspec --allow-warnings --use-libraries
```

### 4.8 pod搜索

```
pod search EHIHiCarBluetooth
```

如果远程验证通过，但是搜索不到，是因为没有添加进`pod search`缓存文件，删掉缓存重建即可！命令如下：

```
# 切换到CocoaPods目录
cd ~/Library/Caches/CocoaPods/
# 查看该目录下有Pods和search_index.json两个文件
ls

# 删除缓存文件
rm search_index.json

# 重新搜索
pod search EHIHiCarBluetooth
```

## 五、使用

`Podfile`文件如下：

```
# CocoaPods官方spec仓库
source 'https://github.com/CocoaPods/Specs.git'
# 私有spec仓库
source 'http://私有库地址/EHILibraryiOS/EHILibrarySpecs.git'

platform :ios, '8.0'
target 'TextPodBluetooth' do

# 私有库
pod 'EHIHiCarBluetooth','~> 0.1.0'

end
```

## 六、参考

- [cocoapods进阶](https://www.zybuluo.com/qidiandasheng/note/392639)
- [CocoaPods进阶：详解私有库制作](https://juejin.im/post/5b82ca46518825430d26bc09)
- [创建自己的本地私有pod（实践）](https://www.jianshu.com/p/103a6f0bf3a4)
- [使用cocoapods打包静态库(依赖私有库，开源库，私有库又包含静态库)](https://www.jianshu.com/p/9096a2eb2804)
- [通过CocoaPods打包framework](https://www.jianshu.com/p/e744b56d57ea)







