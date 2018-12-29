# iOS MVVM 使用要点

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

  - [一、前言](#%E4%B8%80%E5%89%8D%E8%A8%80)
  - [二、MVC](#%E4%BA%8Cmvc)
  - [三、 MVVM](#%E4%B8%89-mvvm)
    - [3.1 基本概念和注意事项](#31-%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5%E5%92%8C%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)
    - [3.2 使用要点](#32-%E4%BD%BF%E7%94%A8%E8%A6%81%E7%82%B9)
    - [3.3 优势](#33-%E4%BC%98%E5%8A%BF)
    - [3.4 弊端](#34-%E5%BC%8A%E7%AB%AF)
  - [四、ReactiveCocoa](#%E5%9B%9Breactivecocoa)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

--------

## 一、前言

网上文章真的很多，在此简单的说一些要点和自己的看法。参考以下这篇文章：

- [iOS 关于MVC和MVVM设计模式的那些事](https://www.jianshu.com/p/caaa173071f3)

## 二、MVC

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/MVC.png)

`MVC` 兼顾 `View` 和 `Model` 的中间处理，还有很多例如网络请求等很多业务逻辑判断，难免不小心就臃肿厚重。

## 三、 MVVM

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/MVVM.png)

### 3.1 基本概念和注意事项

1. `View` 和 `View controller `都视为 `View`；
2. `View `不能直接引用 `Model`，而是引用 `ViewModel`；
3. `ViewModel` 引用 `Model`，但反过来不行（**任何视图本身的引用都不应该放在 ViewModel 中**）；
ViewModel 是一个放置用户输入验证逻辑，视图显示逻辑，发起网络请求和其他代码的地方。
4. 使用 `MVVM` 会轻微的增加代码量，但总体上减少了代码的复杂性。

### 3.2 使用要点

1. `MVVM` 可以兼容你当下使用的 `MVC` 架构；
2. MVVM 增加你的应用的可测试性；
3. MVVM 配合一个绑定机制效果最好（PS：`ReactiveCocoa`）；
4. `ViewController` 尽量不涉及业务逻辑，让 `ViewModel` 去做这些事情；
5. `ViewController` 只是一个中间人，接收 `View` 的事件、调用  `ViewModel` 的方法、响应 `ViewModel` 的变化；
6. `ViewModel` 绝对不能包含视图 `View`（UIKit.h），不然就跟 View 产生了耦合，不方便复用和测试；
7. `ViewModel` 之间可以有依赖；
8. `ViewModel` 避免过于臃肿，否则重蹈 `Controller` 的覆辙，变得难以维护。

### 3.3 优势

**低耦合：**`View` 可以独立于 `Model` 变化和修改。

**可重用性：**
可以把视图布局放在一个 `ViewModel` 里面，让很多 `View` 重用这段视图逻辑；
也可以把视图布局仍放在 `View` 里，多个 `ViewModel` 公用一个布局一致的 `View` 。

**独立开发：**开发人员可以专注于业务逻辑和数据的开发 `ViewModel`，设计人员可以专注于页面设计。

**可测试：**通常界面是比较难于测试的，而 `MVVM` 模式可以针对 `ViewModel` 来进行测试。

### 3.4 弊端

**Bug 很难被定位：**

首先，很多人觉得这样查找 `bug` 更加麻烦了，因为毕竟要多跳转一层 `ViewModel` 。

但是当你熟练使用 `MVVM` 后，你找到 `View` 使用的 `ViewModel`，然后清楚的定位数据和逻辑问题在 `ViewModel` 中。

这里就要强调上面说的一定要层次分明的使用 `ViewModel`，不能是 `ViewModel` 中引用 `View`，`View` 中又包含该放在 `ViewModel` 中的逻辑，这样的话 bug 自然难以定位了。

**ReactiveCocoa：**

`MVVM` 一致强调和 `RAC` 同时使用会更加方便。

**不使用 RAC ：**`ViewModel` 处理完数据需要刷新 `View` 等操作都是通过 `Block` 回调来实现，不免是有些麻烦的。

**使用 RAC ：**在 `View` 初始化的时候就和 `ViewModel` 进行绑定，只要 `ViewModel` 的数据有所变化，便自动更新 `View` 。

**RAC 缺点：**数据绑定使得一个位置的 `Bug` 被快速传递到别的位置，要定位原始出问题的地方就变得不那么容易了。


## 四、ReactiveCocoa

- [最快让你上手ReactiveCocoa之基础篇](http://www.jianshu.com/p/87ef6720a096#)

- [最快让你上手ReactiveCocoa之进阶篇](http://www.jianshu.com/p/e10e5ca413b7) 

响应式编程思想：不需要考虑调用顺序，只需要知道考虑结果，类似于蝴蝶效应，产生一个事件，会影响很多东西，这些事件像流一样的传播出去，然后影响结果，借用面向对象的一句话，万物皆是流。
代表：KVO运用。



ReactiveCocoa 常见宏：
1. RAC(TARGET, [KEYPATH, [NIL_VALUE]])：用于给某个对象的某个属性绑定。
``` bash  
 // 只要文本框文字改变，就会修改label的文字
RAC(self.labelView,text) = _textField.rac_textSignal; 
```
2. RACObserve(self, name)：监听某个对象的某个属性，返回的是信号。
``` bash  
[RACObserve(self.view, center) subscribeNext:^(id x) {
	NSLog(@"%@",x);
}];
```
