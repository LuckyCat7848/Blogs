# iOS常量使用extern和static原理探究，为什么Defines中默认推荐使用extern?

## 一、场景需求

我们在项目架构中，每个模块下都会需要一个相关定义文件。例如：注册、登录、忘记密码为一个`Account`模块，模块下的枚举、宏定义、通知、数据存储 Key 等都放在一个`EHIAccountDefines`文件中。对于宏定义、通知、数据存储 Key 等常量的定义使用我们来讨论一下。

## 二、使用方法

### 2.1 宏定义

```
// EHIAccountDefines.h
#define kEHIChangedPasswordNotification @"UserChangedPassword";
```

最省事的方式就是宏定义了，但是宏定义本质只是文本替换。在预编译器影响速度、不能声明变量类型、不能使用 const 修饰导致使用中值可以更改，所以常量的定义不推荐。一般用于方法的定义（方法也可以使用内联函数`inline`代替一部分，具体使用不多赘述）。

### 2.2 static

```
// EHIAccountDefines.h
static NSString *const kEHILogoutNotification = @"UserDidLogout";
```

这种写法也是很常见的，不止是在这个场景，也有很多用在`.m`中定义一个动画时长、`cell`复用池名称的定义等。

### 2.3 extern

```
// EHIAccountDefines.h
extern NSString *const kEHILoginNotification;

// EHIAccountDefines.m
NSString *const kEHILoginNotification = @"UserDidLogin";
```

> 排除`#define`的写法。`extern`是各个博客和书上的默认写法，没有太多对此的解释，好像理应如此。而实际上`static`的话只需要`.h`文件即可，更加方便。所以这次探究一下为什么还都默认使用`extern`呢？

## 三、探究

### 3.1 举个例子

在`.pch`引用的全局的一个`Defines`文件中写下：

```
// .h
extern NSString *const kEHIConstExternStr;

static NSString *const kEHIConstStaticStr = @"一个静态字符串";
```
```
// .m
NSString *const kEHIConstExternStr = @"一个外部字符串";
```

在几个`ViewController`的`viewDidLoad`中使用这两个字符串（注意是分别几个文件中写，不是基类）：

```
NSLog(@"%@ %@：%p, %x", self.className, kEHIConstExternStr, kEHIConstExternStr, &kEHIConstExternStr);
NSLog(@"%@ %@：%p, %x", self.className, kEHIConstStaticStr, kEHIConstStaticStr, &kEHIConstStaticStr);
```

> `%p`打印的是这个字符串的值的地址，后面`%x`是存放这个变量的地址。
> **只有值类型是直接放的值，引用类型放的都是地址。**字符串是引用类型，故而它地址存放的是它的值地址。

### 3.2 输出结果

```
[1982:519479] AppDelegate 一个外部字符串：0x106715780, 6692cd8
[1982:519479] AppDelegate 一个静态字符串：0x1066e2020, 6669410

[1982:519479] EHIHomeContantViewController 一个外部字符串：0x106715780, 6692cd8
[1982:519479] EHIHomeContantViewController 一个静态字符串：0x1066e2020, 667b080

[1982:519479] EHIHomeViewController 一个外部字符串：0x106715780, 6692cd8
[1982:519479] EHIHomeViewController 一个静态字符串：0x1066e2020, 667d530
```

查看输出结果，可以得知：

**`extern`**：变量地址和值地址都是同一个。
**`static`**：值地址是同一个，但是变量地址是不同的。

那么 static 在所有地方引用，是在所有地方都分配一个变量地址呢，还是只有使用到才会分配呢？**反编译查看：**
``` 
static NSString *const kShowGuideKey = @"kShowGuideKey01";
```
**static** 的变量地址是变的，但是在用到的地方才会创建。三个文件里有用到，就会自动生成三个变量，分别是`_kShowGuideKey`、`_kShowGuideKey_101980440`、 `_kShowGuideKey_1019949c0`，都指向同一个字符串`@“kShowGuideKey01”`值。
编译的时候第一次用到是原名加下划线前缀，只有再有用到会追加地址最为后缀区分。

**extern** 的不管多少个地方用到，编译时只有一个下划线前缀的变量，因为 extern 的只有一个`.m`有这个定义。

### 3.3 分析

**`extern`**：全局只有一个变量，对一个常量字符串的引用，只会被初始化一次。
**`static`**：全局有多个变量，每个引用的地方的变量都会初始化一次，分配内存空间，对一个常量字符串的引用（所以很多博客和书根本就没提这样写）。

#### 3.3.1 static 的原理

*而为什么会有这个结论呢？static 修饰的变量为什么多次分配内存呢？*

上面的使用，如果既不用 extern 修饰，也不用 static 修饰呢？答案是报错，因为重复定义了。
我们先了解下 .h 的作用：

> `.h`文件可以说没有用，只是给外部看的。`h`中是实际上无法定义变量的，编译`.m`的时候会把`.h`中的内容在`.m`中展开后编译才会真正的生成一个变量。

所以，没有任何修饰符的话相当于多个类中都定义了同一个变量名，就重复定义报错了。

然而，这里加上`static`的话，就不会报错，只是每个文件使用的变量不一样。也就是说 static 限制了该变量作用域在每个使用它的文件内。

我们可以得到结论：

> **`static`：限制作用域。**

（这里就不扩展说它修饰局部变量、全局变量、函数的作用了~~~ ）

#### 3.3.2 extern 的原理

*接下来，为什么 extern 可以保持变量和常量一致呢？*

我们看个例子，日常开发中在 .m 中使用 static 的情况很多，例如：

```
static NSString *const kEHIShowGuideKey = @"EHIShowGuideKey";
```

如果不加 static 修饰符呢？

```
// ClassA.m
NSString *const kEHIShowGuideKey = @"EHIShowGuideKey";
```

系统会自动给我们加上 extern 修饰为外部变量。按照上面说的 static 限制作用域的原理，我们在其他地方就`不能重复定义`这个变量名称了。但是它现在是一个外部变量，我们可以在其它地方`使用`这个变量。

如果其他地方使用，我们可以这么用：

```
// ClassB.m
extern NSString *const kEHIShowGuideKey;
```

ClassB 在使用时，声明我在使用一个外部变量。但是这样就要每个使用该字符串的时候都去声明下。

所以常规的写法还是如下：
```
// ClassA.h
extern NSString *const kEHIShowGuideKey;

// ClassA.m
NSString *const kEHIShowGuideKey = @"EHIShowGuideKey";
```

我们可以得到结论：

> **`extern`：外部的，作用就是修饰后面的内容是由外部定义的而已。**

## 四、总结

终于把思路理清楚写完了！😭

了解了原理再看各种情况就清晰明了了。还没有一下就明白的小伙伴自己写写例子、消化下哈。
