# iOS开发规范 (For 一嗨租车)

[toc]

------














  
  
 
## 一、前言

  * 为了让代码更加整洁，团队配合更加顺畅，制定开发规范文档，请 iOS 开发同学仔细阅读本文档，并在日常开发过程中遵守本文档中的约定。本规范文档结合 [Apple's Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)、[Google's OC Guide](https://google.github.io/styleguide/objcguide.xml) ，以及行业内比较知名的编码规范和开发团队的经验总结而制定，例如 [Github's Guide](https://github.com/github/objective-c-style-guide)、[Raywenderlich's Guide](https://github.com/raywenderlich/objective-c-style-guide) ，有不完善和不合理的地方后期会持续修改增删。
 
## 二、项目目录  	

```
|- AppDelegate	// 目录下放的是AppDelegate.h(.m)文件，是整个应用的入口文件，所以单独拿出来
|- Macro 		// 整个应用会用到的宏定义
	|- EHIAppMacro.h          // app相关的宏定义
    |- EHINotificationMacro.h // 通知相关的宏定义
    |- EHIEventMacro.h        // 统计key的宏定义
    |- EHIBasicDesignMacro.h  // 适配、颜色
    |- EHIVenderMacro.h       // 第三方的key
    |- EHIUtilsMacro.h        // 一些方便使用的宏定义
|- Features		// 功能模块目录     
    |- Load     // 主页加载之前的欢迎引导页
    |- Login
		|- EHILoginMacro.h // 枚举、提示语、数据存储ID（每个模块都根据需要定义）      
	    |- 注册
	    |- 登录
	    |- 忘记密码
    |- UserCenter
	    |- EHIUserCenterMacro.h 
        |- ViewControllers
        |- ViewModels
        |- Models       
        |- Views 
        |- 设置
        |- 我的账户
        |- 会员
	    |- 余额
	    ...
	...
|- Tools        // 工具类、Categories
    |- UIButton // 系统类的Category建一个独立的文件夹
	    |- UIButton+EdgeInsets
	    |- UIButton+EHIExtension
		...
    |- EHILoading // 各种功能，都以文件夹形式存在 
    ...
|- Helpers      // 帮助类。包含网络、数据库、定位等操作类的封装和实现
	|- NetWork
	|- Data
    |- Location
    ...
|- Vendors		// 第三方的类库/SDK。部分需要修改或者不支持cocoapod的第三方的框架引入
|- Resources  	// 资源文件。包含图片、声音、文件（.plist/.json/.ttc）
	|- GIFImages
	|- ProjectCer
	...
|- Supporting Files // Xcode自带文件夹
	|- Images.xcassets // 工程默认图片存放文件夹,可以在该下按不同模块创建文件夹存放对应的图片
```

## 三、代码格式

### 3.1 语言

应该使用 US 英语。

应该:
``` 
UIColor *myColor = [UIColor whiteColor];
```
不应该:
```
UIColor *myColour = [UIColor whiteColor];
```

### 3.2 空格和换行

1. 使用 `Tab` 符号，而非空格。首行缩进一个 Tab ，即 4 个空格；
2. 使用换行符结束一个文件；
3. 方法大括号和其他大括号（`if`/`else`/`switch`/`while`/`for` 等）总是在同一行语句打开但在新行中关闭；
4. 合理的使用空格将代码划分为逻辑块，例如 `if` 语句、  `for` 循环、三元操作符。

应该：

```
for (int i = 0; i < 10; i++) {
}
```

不应该：

```
for(int i=0;i<10;i ++){
}
```

5. 在方法之间应该有且只有一行，这样有利于在视觉上更清晰和更易于组织。在方法内的空白应该分离功能，但通常都抽离出来成为一个新方法；
6. 优先使用 auto-synthesis 。但如果有必要，`@synthesize` 和 `@dynamic` 应该在实现中每个都声明新的一行。

### 3.3 每一行的最大长度

过长的一行代码将会导致可读性问题，一些规范建议为 80 。

一些常用的如 UITableView 的方法长度就大于 80 ，建议注释上强制要求，方法上不强制。

可以在 **Xcode > Preferences > Text Editing > Page guide at column** 中将最大行长设置为  80 。 

### 3.4 Case语句

- 大括号在 `case` 语句中并不是必须的，除非编译器强制要求；
- 当一个 `case` 语句包含多行代码时，大括号应该加上；
- `break` 放在大括号 `}` 外。

```
switch (condition) {
  case 1:
    // ...
    break;
  case 2: {
    // ...
    // Multi-line example using braces
 }
    break;
  case 3:
    // ...
    break;
  default: 
    // ...
    break;
}
```

有很多次，当相同代码被多个 cases 使用时，一个 `fall-through` 应该被使用。一个 `fall-through` 就是在 `case` 最后移除 `break` 语句，这样就能够允许执行流程跳转到下一个 `case` 值。为了代码更加清晰，一个 fall-through 需要注释一下。

```
switch (condition) {
  case 1:
    // ** fall-through! **
  case 2:
    // code executed for values 1 and 2
    break;
  default: 
    // ...
    break;
}
```

当在 switch 使用枚举类型时，`default` 是不需要的。例如：

```
RWTLeftMenuTopItemType menuType = RWTLeftMenuTopItemMain;

switch (menuType) {
  case RWTLeftMenuTopItemMain:
    // ...
    break;
  case RWTLeftMenuTopItemShows:
    // ...
    break;
  case RWTLeftMenuTopItemSchedule:
    // ...
    break;
}
```

### 3.5 布尔值

1. Objective-C 使用 `YES` 和 `NO` 。因为 `true` 和 `false` 应该只在 CoreFoundation，C 或 C++ 代码使用；
2. 既然 `nil` 解析成 `NO` ，所以没有必要在条件语句比较；
3. 不要拿某样东西直接与 `YES` 比较，因为 `YES` 被定义为 1，而一个 `BOOL` 能被设置为 8 位。

这是为了在不同文件保持一致性和在视觉上更加简洁而考虑。

应该:

```
if (someObject) {}
if (![anotherObject boolValue]) {}
```

不应该:

```
if (someObject == nil) {}
if ([anotherObject boolValue] == NO) {}
if (isAwesome == YES) {} // Never do this.
if (isAwesome == true) {} // Never do this.
```

如果 `BOOL` 属性的名字是一个形容词，属性就能忽略“ is ”缀，但要指定 get 访问器的惯用名称。例如：

```
@(工作日常)property (nonatomic, assign, getter=isEditable) BOOL editable;
```

文字和例子从这里引用 [Cocoa Naming Guidelines](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE) 。

### 3.6 条件语句

条件语句主体为了防止出错应该使用大括号包围，即使条件语句主体能够不用大括号编写（如，只用一行代码）。

这些错误包括添加第二行代码和期望它成为 `if` 语句。例如 [Single statement if block - braces or no?](https://softwareengineering.stackexchange.com/a/16530 ) 中可能发生在 `if` 语句里面一行代码被注释了，然后下一行代码不知不觉地成为 `if` 语句的一部分。

除此之外，这种风格与其他条件语句的风格保持一致，所以更加容易阅读。

应该:

```
if (!error) {
    return success;
}
```

不应该:

```
if (!error)
    return success;
```

或

```
if (!error) return success;
```

#### 3.6.1 三元操作符

1. 当需要提高代码的清晰性和简洁性时，三元操作符 `?:` 才会使用。单个条件求值常常需要它；
2. 多个条件求值时，如果使用 `if` 语句或重构成实例变量时，代码会更加易读；
3. 一般来说，最好使用三元操作符是在根据条件来赋值的情况下。

`Non-boolean` 的变量与某东西比较，加上括号 `()` 会提高可读性。如果被比较的变量是 `boolean` 类型，那么就不需要括号。

应该:

```
NSInteger value = 5;
result = (value != 0) ? x : y;

BOOL isHorizontal = YES;
result = isHorizontal ? x : y;
```

不应该:

```
result = a > b ? x = c > d ? c : d : y;
```

### 3.7 Init 方法和类构造方法（instancetype）

当初始化方法和类构造方法被使用时，它应该返回类型是 `instancetype` 而不是 `id` 。这样确保编译器正确地推断结果类型。

```
- (instancetype)init {
  self = [super init];
  if (self) {
    // ...
  }
  return self;
}
```

```
@interface Airplane
+ (instancetype)airplaneWithType:(RWTAirplaneType)type;
@end
```

关于更多 `instancetype` 信息，请查看 [NSHipster.com](http://nshipster.com/instancetype/) 。

### 3.8 CGRect函数

当访问 `CGRect` 里的 `x`/`y`/`width`/`height` 时，应该使用 `CGGeometry` 函数而不是直接通过结构体来访问。引用 Apple 的 CGGeometry ：

>在这个参考文档中所有的函数，接受 `CGRect` 结构体作为输入，在计算它们结果时隐式地标准化这些 `rectangles` 。因此，你的应用程序应该避免直接访问和修改保存在 `CGRect` 数据结构中的数据。相反，使用这些函数来操纵 `rectangles` 和获取它们的特性。

应该:

```
CGRect frame = self.view.frame;

CGFloat x = CGRectGetMinX(frame);
CGFloat y = CGRectGetMinY(frame);
CGFloat width = CGRectGetWidth(frame);
CGFloat height = CGRectGetHeight(frame);
CGRect frame = CGRectMake(0.0, 0.0, width, height);
```

不应该:

```
CGRect frame = self.view.frame;

CGFloat x = frame.origin.x;
CGFloat y = frame.origin.y;
CGFloat width = frame.size.width;
CGFloat height = frame.size.height;
CGRect frame = (CGRect){ .origin = CGPointZero, .size = frame.size };
```

### 3.9 黄金路径（使用 return）

当使用条件语句编码时，不要嵌套 `if` 语句，多个返回语句更 OK 。

应该:

```
- (void)someMethod {
  if (![someOther boolValue]) {
	return;
  }

  // Do something important
}
```

不应该:

```
- (void)someMethod {
  if ([someOther boolValue]) {
    // Do something important
  }
}
```

### 3.10 错误处理

当方法通过引用来返回一个错误参数，判断返回值而不是错误变量。

应该:

```
NSError *error;
if (![self trySomethingWithError:&error]) {
  // Handle Error
}
```

不应该:

```
NSError *error;
[self trySomethingWithError:&error];
if (error) {
  // Handle Error
}
```

在成功的情况下，有些 Apple 的 APIs 记录垃圾值（garbage values）到错误参数（如果 non-NULL），那么判断错误值会导致 false 负值和 crash 。

### 3.11 Block

根据 `block` 的长度，有不同的书写规则：

- 较短的 block 可以写在一行内（不强制）；
- 如果分行显示的话，block 的右括号 `}` 应该和调用 block 那行代码的第一个非空字符对齐；
- block 内的代码采用 4 个空格的缩进；
- 如果 block 过于庞大，应该单独声明成一个变量来使用；
- `^` 和 `(` 之间，`^` 和 `{` 之间都没有空格，参数列表的右括号 `)` 和 `{` 之间有一个空格。

```
// 较短的block可以写在一行内
    [operation setCompletionBlock:^{ [self onOperationDone]; }];
    
    // 分行书写的block，内部使用4空格缩进
    [operation setCompletionBlock:^{
        [self.delegate newDataAvailable];
    }];
    
    // 使用C语言API调用的block遵循同样的书写规则
    dispatch_async(_fileIOQueue, ^{
        NSString *path = [self sessionFilePath];
        if (path) {
            // …
        }
    });
    
    // 较长的block关键字可以缩进后在新行书写，注意block的右括号`}`和调用block那行代码的第一个非空字符对齐
    // (Xcode本身对齐规则就是这样，平时很少注意到)
    [[SessionService sharedService]
     loadWindowWithCompletionBlock:^(SessionWindow *window) {
         if (window) {
             [self windowDidLoad:window];
         } else {
             [self errorLoadingWindow];
         }
     }];
    
    // 较长的block参数列表同样可以缩进后在新行书写
    [[SessionService sharedService]
     loadWindowWithCompletionBlock:
     ^(SessionWindow *window) {
         if (window) {
             [self windowDidLoad:window];
         } else {
             [self errorLoadingWindow];
         }
     }];
    
    // 庞大的block应该单独定义成变量使用
    void (^largeBlock)(void) = ^{
        // …
    };
    [_operationQueue addOperationWithBlock:largeBlock];
    
    // 在一个调用中使用多个block，注意到他们不是像函数那样通过`:`对齐的，而是同时进行了4个空格的缩进
    [myObject doSomethingWith:arg1
        firstBlock:^(Foo *a) {
            // …
        }
        secondBlock:^(Bar *b) {
            // …
        }];
```

另外要注意的是：
>**block 使用的时候判断一下，再回调，防止未实现，导致 crash。**

```
if (completion) {
	completion(true);
}
```

### 3.12 数据结构的语法糖

应该使用可读性更好的语法糖来构造 `NSArray`，`NSDictionary` 等数据结构，避免使用冗长的 `alloc`、`init` 方法。

例子：

```
NSArray *array = @[[foo description], [bar description]];
NSDictionary *dict = @{NSForegroundColorAttributeName: [NSColor redColor]};
```

如果构造代码不写在一行内，右括号 `]` 或者 `}` 写在新的一行，语句短的和长的缩进如下：

```
// 短的
NSArray *array = @[
                   @"This",
                   @"is",
                   @"an",
                   @"array"
                   ];

// 长的
NSDictionary *dictionary = @{
    NSFontAttributeName : [UIFont fontWithName:@"Helvetica-Bold" size:12],
    NSForegroundColorAttributeName : fontColor
};
```

构造字典时，字典的 `Key` 和 `Value` 与中间的冒号 `:` 都要留有一个空格，多行书写时，也可以将冒号 `:` 对齐：

```
// 正确，冒号`:`前后留有一个空格
NSDictionary *option1 = @{
    NSFontAttributeName : [UIFont fontWithName:@"Helvetica-Bold" size:12],
    NSForegroundColorAttributeName : fontColor
};

// 正确，按照冒号`:`来对齐
NSDictionary *option2 = @{
    NSFontAttributeName            : [NSFont fontWithName:@"Arial" size:12],
    NSForegroundColorAttributeName : fontColor
};

//错误，冒号前应该有一个空格
NSDictionary *wrong = @{
                        AKey: @"b",
                        BLongerKey: @"c",
                        };

//错误，每一个元素要么单独成为一行，要么全部写在一行内
NSDictionary *alsoWrong = @{AKey : @"a",
                            BLongerKey : @"b"};
```

## 四、命名规范

### 4.1 基本原则

#### 4.1.1 清晰性

命名应该尽可能的清晰和简洁，但在 Objective-C 中，清晰比简洁更重要。由于 Xcode 强大的自动补全功能，我们不必担心名称过长的问题。

```
// 清晰 
insertObject:atIndex:

// 不清晰，insert的对象类型和at的位置属性没有说明 
insert:at:

// 清晰 
removeObjectAtIndex:

// 不清晰，remove的对象类型没有说明，参数的作用没有说明 
remove: 
```

不要使用单词的简写，拼写出完整的单词：

```
// 清晰 
destinationSelection 
setBackgroundColor:

// 不清晰，不要使用简写 
destSel 
setBkgdColor: 
```

然而，有部分单词简写在 Objective-C 编码过程中是非常常用的，以至于成为了一种规范，这些简写可以在代码中直接使用，下面列举了部分：

```
alloc == Allocate       max == Maximum 
alt == Alternate        min == Minimum 
app == Application      msg == Message 
calc == Calculate       nib == Interface Builder archive 
dealloc == Deallocate   init == Initialize  
func == Function        rect == Rectangle 
horiz == Horizontal     vert == Vertical 
info == Information     temp == Temporary 
```

命名方法或者函数时要避免歧义：

```
// 有歧义，是返回sendPort还是send一个Port？ 
sendPort

// 有歧义，是返回一个名字属性的值还是display一个name的动作？ 
displayName
```

#### 4.1.2 一致性

整个工程的命名风格要保持一致性。

比如不同 Model 中表示性别应该用一样的名字，比如我们需要表示性别，不可以有的用 gender ，有的用 sex 。

或者不同类中完成相似功能的方法应该叫一样的名字，比如我们总是用 count 来返回集合的个数，不能在 A 类中使用 count 而在 B 类中使用 getNumber 。


### 4.2 使用前缀

- 前缀由大写的字母缩写组成，比如 Cocoa 中 NS 前缀代表 Founation 框架中的类， IB 则代表 Interface Builder 框架。

- 可以在为类、协议、函数、常量以及 typedef 宏命名的时候使用前缀，但注意不要为成员变量或者方法使用前缀，因为他们本身就包含在类的命名空间内。 

- 命名前缀的时候不要和苹果 SDK 框架冲突。

### 4.3 图片资源的命名

规则如下：

- 用英文命名，不用拼音；
- 每一部分用下划线分隔；
- 图片名中两倍图在名字最后要加 `@2x`，三倍图在名字最后要加 `@3x`。

格式如下：

> 逻辑分类 _ 表现内容 _ 图片状态


某个模块用到的，以模块作为第一部分，例子：

```
tabbar_chat // tabbar聊天模块正常状态切图
tabbar_chat_selected // tabbar聊天模块选中状态切图

chat_switchCamera // 聊天模块中拍摄界面的切换摄像头切图
```

有的切图是整个 App 用到的，例子：

```
public_back // 返回切图
public_close // 关闭切图
```

### 4.4 类和协议（ Class & Protocol ） 的命名

类名以大写字母开头，应该包含一个名词来表示它代表的对象类型如 Model 。
同时可以加上统一的前缀，比如 NSString、NSArray 等等以 NS 开头。 我们项目使用 `EHI` 。

而协议名称应该清晰地表示它所执行的行为，而且要和类名区别开来，所以通常使用 `ing` 词尾来命名一个协议，比如 NSCopying、NSLocking 。

### 4.5 委托（Delegate）的命名

当特定的事件发生时，对象会触发它注册的委托方法。委托是 Objective-C 中常用的传递消息的方式。委托有它固定的命名范式。

委托的命名参考类名，通常使用 `Delegate` 词尾，比如 UITableViewDelegate、UIScrollViewDelegate。

一个委托方法的第一个参数是触发它的对象，第一个关键词是触发对象的类名，除非委托方法只有一个名为 sender 的参数：

```
// 第一个关键词为触发委托的类名
- (BOOL)tableView:(NSTableView *)tableView shouldSelectRow:(int)row;
- (BOOL)application:(NSApplication *)sender openFile:(NSString *)filename;

// 当只有一个"sender"参数时可以省略类名
- (BOOL)applicationOpenUntitledFile:(NSApplication *)sender;
```

根据委托方法触发的时机和目的，使用 should , will , did 等关键词：

```
- (void)browserDidScroll:(NSBrowser *)sender;

- (NSUndoManager *)windowWillReturnUndoManager:(NSWindow *)window;

- (BOOL)windowShouldClose:(id)sender;
```

###4.6 头文件的命名（Headers）

源码的头文件名应该清晰地暗示它的功能和包含的内容：

- 如果头文件内只定义了单个类或者协议，直接用类名或者协议名来命名头文件，比如 `NSLocale.h` 定义了 `NSLocale` 类。

- 如果头文件内定义了一系列的类、协议、类别，使用其中最主要的类名来命名头文件，比如 `NSString.h` 定义了 `NSString` 和 `NSMutableString`。

- 每一个 Framework 都应该有一个和框架同名的头文件，包含了框架中所有公共类头文件的引用，比如 `Foundation.h`。

- Framework 中有时候会实现在别的框架中类的类别扩展，这样的文件通常使用 `被扩展的框架名+Additions` 的方式来命名，比如 `NSBundleAdditions.h`。

### 4.7 方法的命名

在方法签名中，应该在方法类型（`-`/`+` 符号）之后有一个空格。在方法各个段之间应该也有一个空格（符合Apple的风格）。在参数之前应该包含一个具有描述性的关键字来描述参数。

方法一般以小写字母打头，每一个后续的单词首字母大写，方法名中不应该有标点符号（包括下划线），有两个例外：

- 可以用一些通用的大写字母缩写打头方法，比如 PDF、TIFF 等。
- 可以用带下划线的前缀来命名私有方法或者类别中的方法（如 MJRefresh ）。

如果方法表示让对象执行一个动作，使用动词打头来命名，注意不要使用 do，does 这种多余的关键字，动词本身的暗示就足够了：

```
// 动词打头的方法表示让对象执行一个动作
- (void)invokeWithTarget:(id)target;
- (void)selectTabViewItem:(NSTabViewItem *)tabViewItem;
```

如果方法是为了获取对象的一个属性值，直接用属性名称来命名这个方法，注意不要添加 get 或者其他的动词前缀：

```
// 正确，使用属性名来命名方法
- (NSSize)cellSize;

// 错误，添加了多余的动词前缀
- (NSSize)calcCellSize;
- (NSSize)getCellSize;
```

对于有多个参数的方法，务必在每一个参数前都添加关键词，关键词应当清晰说明参数的作用。

`and` 这个词的用法应该保留。它不应该用于多个参数来说明，就像 `initWithWidth:height` 以下这个例子：

应该:

```
- (void)setExampleText:(NSString *)text image:(UIImage *)image;
- (void)sendAction:(SEL)aSelector to:(id)anObject forAllCells:(BOOL)flag;
- (id)viewWithTag:(NSInteger)tag;
- (instancetype)initWithWidth:(CGFloat)width height:(CGFloat)height;
```

不应该:

```
-(void)setT:(NSString *)text i:(UIImage *)image; // 关键词不清晰
- (void)sendAction:(SEL)aSelector :(id)anObject :(BOOL)flag; // 无关键词
- (id)taggedView:(NSInteger)tag; // 关键词的作用不清晰
- (instancetype)initWithWidth:(CGFloat)width andHeight:(CGFloat)height;
- (instancetype)initWith:(int)width and:(int)height;  // Never do this.
```

方法的参数命名也有一些需要注意的地方：

- 和方法名类似，参数的第一个字母小写，后面的每一个单词首字母大写。
- 不要再方法名中使用类似 `pointer`、`ptr` 这样的字眼去表示指针，参数本身的类型足以说明。
- 不要使用只有一两个字母的参数名。
- 不要使用简写，拼出完整的单词。

下面列举了一些常用参数名：

```
...action:(SEL)aSelector
...alignment:(int)mode
...atIndex:(int)index
...content:(NSRect)aRect
...doubleValue:(double)aDouble
...floatValue:(float)aFloat
...font:(NSFont *)fontObj
...frame:(NSRect)frameRect
...intValue:(int)anInt
...keyEquivalent:(NSString *)charCode
...length:(int)numBytes
...point:(NSPoint)aPoint
...stringValue:(NSString *)aString
...tag:(int)anInt
...target:(id)anObject
...title:(NSString *)aString
```

### 4.8 存取方法（Accessor Methods）的命名

存取方法是指用来获取和设置类属性值的方法，属性的不同类型，对应着不同的存取方法规范：

```
// 属性是一个名词时的存取方法范式
- (type)noun;
- (void)setNoun:(type)aNoun;
// 例子
- (NSString *)title;
- (void)setTitle:(NSString *)aTitle;

// 属性是一个形容词时存取方法的范式
- (BOOL)isAdjective;
- (void)setAdjective:(BOOL)flag;
// 例子
- (BOOL)isEditable;
- (void)setEditable:(BOOL)flag;

// 属性是一个动词时存取方法的范式
- (BOOL)verbObject;
- (void)setVerbObject:(BOOL)flag;
// 例子
- (BOOL)showsAlpha;
- (void)setShowsAlpha:(BOOL)flag;
```

命名存取方法时不要将动词转化为被动形式来使用：

```
// 正确
- (void)setAcceptsGlyphInfo:(BOOL)flag;
- (BOOL)acceptsGlyphInfo;

// 错误，不要使用动词的被动形式
- (void)setGlyphInfoAccepted:(BOOL)flag;
- (BOOL)glyphInfoAccepted;
```

可以使用 `can`、`should`、`will` 等词来协助表达存取方法的意思，但不要使用 `do` 和 `does`：

```
// 正确
- (void)setCanHide:(BOOL)flag;
- (BOOL)canHide;
- (void)setShouldCloseDocument:(BOOL)flag;
- (BOOL)shouldCloseDocument;
```

```
// 错误，不要使用`do`或者`does`
- (void)setDoesAcceptGlyphInfo:(BOOL)flag;
- (BOOL)doesAcceptGlyphInfo;
```

为什么 Objective-C 中不适用 `get` 前缀来表示属性获取方法？因为 get 在 Objective-C 中通常只用来表示从函数指针返回值的函数：

```
// 三个参数都是作为函数的返回值来使用的，这样的函数名可以使用`get`前缀
- (void)getLineDash:(float *)pattern count:(int *)count phase:(float *)phase;
```

### 4.9 集合操作类方法（Collection Methods）的命名

有些对象管理着一系列其它对象或者元素的集合，需要使用类似`增删查改`的方法来对集合进行操作，这些方法的命名范式一般为：

```
// 集合操作范式
- (void)addElement:(elementType)anObj;
- (void)removeElement:(elementType)anObj;
- (NSArray *)elements;

// 例子
- (void)addLayoutManager:(NSLayoutManager *)obj;
- (void)removeLayoutManager:(NSLayoutManager *)obj;
- (NSArray *)layoutManagers;
```

注意，如果返回的集合是无序的，使用 `NSSet` 来代替 `NSArray`。如果需要将元素插入到特定的位置，使用类似于这样的命名：

```
- (void)insertLayoutManager:(NSLayoutManager *)obj atIndex:(int)index;
- (void)removeLayoutManagerAtIndex:(int)index;
```

如果管理的集合元素中有指向管理对象的指针，要设置成 `weak` 类型以防止引用循环。

下面是 SDK 中 `NSWindow` 类的集合操作方法：

```
- (void)addChildWindow:(NSWindow *)childWin ordered:(NSWindowOrderingMode)place;
- (void)removeChildWindow:(NSWindow *)childWin;
- (NSArray *)childWindows;
- (NSWindow *)parentWindow;
- (void)setParentWindow:(NSWindow *)window;
```

### 4.10 函数（Functions）的命名

函数的命名和方法有一些不同，主要是：

 - 函数名称一般带有缩写前缀，表示方法所在的框架。
 - 前缀后的单词以“驼峰”表示法显示，第一个单词首字母大写。

函数名的第一个单词通常是一个动词，表示方法执行的操作：

```
NSHighlightRect
NSDeallocateObject
```

如果函数返回其参数的某个属性，省略动词：

```
unsigned int NSEventMaskFromType(NSEventType type)
float NSHeight(NSRect aRect)
```

如果函数通过指针参数来返回值，需要在函数名中使用Get：

```
const char *NSGetSizeAndAlignment(const char *typePtr, unsigned int *sizep, unsigned int *alignp)
```

函数的返回类型是BOOL时的命名：

```
BOOL NSDecimalIsNotANumber(const NSDecimal *decimal)
```

### 4.11 属性的命名

所有属性特性应该显式地列出来，因为一般都是用 `nonatomic` ，但是默认是 `atomic` 且不频繁使用，所以为了看着整齐，把 `nonatomic` 写在前面。

属性和对象的存取方法相关联，属性的第一个字母小写，后续单词首字母大写，不必添加前缀。属性按功能命名成名词或者动词：

```
// 名词属性
@property (nonatomic, strong) NSString *title;

// 动词属性
@property (nonatomic, assign) BOOL showsAlpha;
```

属性也可以命名成形容词，这时候通常会指定一个带有 `is` 前缀的 `get` 方法来提高可读性：

```
@property (nonatomic, assign, getter=isEditable) BOOL editable;
```

命名实例变量，在变量名前加上 `“_”` 前缀（有些有历史的代码会将 `“_”` 放在后面），其它和命名属性一样：

```
@implementation MyClass {
    BOOL _showsTitle;
}
```

当使用属性时，实例变量应该使用点语法 `self.` 来访问和改变。当你使用点语法时，通过使用 getter 或 setter 方法，属性仍然被访问或修改。想了解更多，阅读[这里](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html)。

但有一个特例：在初始化方法里，实例变量（例如： _variableName ）应该直接被使用来避免 getters/setters 潜在的副作用。

局部变量不应该包含下划线。

### 4.12 对象的命名

对象的命名都统一格式如下：

> 名称 + 类名

应该：

```
UIButton *settingsButton;
```

不应该：

```
UIButton *setBut;
```

### 4.13 通知（Notifications）的命名

通知常用于在模块间传递消息，所以通知要尽可能地表示出发生的事件。

但是请慎用，例如一个页面 FirstViewController 出另一个 SecondViewController ，SecondViewController 给 FirstViewController 传值用 Block 即可。

通知的命名范式是：

> k + [触发通知的类名] + [Did | Will(状态)] + [动作] + Notification

例如系统的都以 NS 开头：

```
NSApplicationDidBecomeActiveNotification
NSWindowDidMiniaturizeNotification
NSTextViewDidChangeSelectionNotification
NSColorPanelColorDidChangeNotification
```

### 4.14 常量、宏、枚举的命名

如果要定义一组相关的常量，尽量使用枚举类型，枚举类型的命名规则和函数的命名规则相同。 建议使用 `NS_ENUM` 和 `NS_OPTIONS` 宏来定义枚举类型：

```
// 定义一个枚举
typedef NS_ENUM(NSInteger, NSMatrixMode) {
    NSRadioModeMatrix,
    NSHighlightModeMatrix,
    NSListModeMatrix,
    NSTrackModeMatrix
};
```

使用匿名枚举定义 bit map ：

```
typedef NS_OPTIONS(NSUInteger, NSWindowMask) {
    NSBorderlessWindowMask      = 0,
    NSTitledWindowMask          = 1 << 0,
    NSClosableWindowMask        = 1 << 1,
    NSMiniaturizableWindowMask  = 1 << 2,
    NSResizableWindowMask       = 1 << 3
};
```

使用 `const` 定义浮点型或者单个的整数型常量，如果要定义一组相关的整数常量，应该优先使用枚举。常量的命名规范和函数相同：

```
static NSString * const EHITableViewCellReuseIdentifier = @"TableViewCellReuseIdentifier";
```

不要使用 `#define` 宏来定义常量，如果是整型常量，尽量使用枚举，浮点型常量，使用 `const` 定义。`#define` 统一格式要加前缀 `k` ：

```
#define kUIColorHex(_hex_) [UIColor colorWithHexString:((__bridge NSString *)CFSTR(#_hex_))]
```

## 五、注释

### 5.1 文件注释

不强制性要求，如果需要，写的清晰易懂即可。这里拿 AFNetWorking 的框架来做模板：

```
// AFHTTPRequestOperation.h
//
// Copyright (c) 2013-2014 AFNetworking (http://afnetworking.com)
//
// Permission is hereby granted, free of charge, to any person obtaining a copy
// of this software and associated documentation files (the "Software"), to deal
// in the Software without restriction, including without limitation the rights
// to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
// copies of the Software, and to permit persons to whom the Software is
// furnished to do so, subject to the following conditions:
//
// The above copyright notice and this permission notice shall be included in
// all copies or substantial portions of the Software.
//
// THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
// IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
// FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
// AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
// LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
// OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
// THE SOFTWARE.
```

### 5.2 方法和属性注释

在 `.h` 文件中采用 Apple 的标准注释风格，好处是可以在引用的地方 `alt + 点击` 自动弹出注释，在写代码调用时也有提示方法或属性的说明，非常方便。

有很多可以自动生成注释格式的插件，推荐使用 [VVDocumenter](https://github.com/onevcat/VVDocumenter-Xcode)： 

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/VVDocumenter.jpg)

在 `.m` 文件中，这种方式参数多的时候占行数多也太啰嗦，可以用如下良好的方式说明即可：

```
/** Stop current preconnecting when application is going to background */
- (void)stopRunning {
}
```

协议、委托的注释要明确说明其被触发的条件：

```
/** Delegate - Sent when failed to init connection, like p2p failed. / 
- (void)initConnectionDidFailed:(IPCConnectHandler *)handler {
}
```

在方法实现中使用 `//` 注释即可，但是后面必须追加一个空格分隔后面的注释语句。

如果在注释中要引用参数名或者方法函数名，使用一对 “ ` ” 将参数或者方法括起来以避免歧义：

```
// Sometimes we need `count` to be less than zero.

// Remember to call `StringWithoutSpaces(“foo bar baz”)`.
```

### 5.3 段注释

```
#pragma mark - 段索引
```

该语句上下各保留一个换行，这个是一个分割线的效果，便于多个方法分类查看。

## 六、编码风格

### 6.1 关于如何在代码块中留回车

这里拿的是 AFNetworking 框架做的一个说明：

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/EnterCodingStandards.jpg)

### 6.2 点符号语法

点语法是一种很方便封装访问方法调用的方式。当你使用点语法时，通过使用 `getter` 或 `setter` 方法，属性仍然被访问或修改。想了解更多，阅读 [这里](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html) 。

点语法应该总是被用来访问和修改属性，因为它使代码更加简洁。`[]` 符号更偏向于用在其他例子。

应该:

```
NSInteger arrayCount = [self.array count];
view.backgroundColor = [UIColor orangeColor];
[UIApplication sharedApplication].delegate;
```

不应该:

```
NSInteger arrayCount = self.array.count;
[view setBackgroundColor:[UIColor orangeColor]];
UIApplication.sharedApplication.delegate;
```

### 6.3 nil、Nil、Null、NSNull 的使用

我们先来看下这些空值的定义：

```
nil:  Defines the id of a null instance，指向一个（实例）对象的空指针 
例如：
NSString *msg = nil;
NSDate *date =nil;
```

```
Nil: Defines the id of a null class，指向一个类的空指针
例如：
Class class = Nil;
```

```
NULL：定义其他类型（基本类型、C类型）的空指针
char *p = NULL;
```

```
NSNull:数组中元素的占位符，数据中的元素不能为nil（可以为空，也就是NSNull），
原因：nil 是数组的结束标志
如果用nil，就会变成
NSArray *array = [NSArray arrayWithObjects:
[[NSObject alloc] init], 
nil,
[[NSObject alloc] init], 
[[NSObject alloc] init], 
nil];，
那么数组到第二个位置就会结束。打印[array count]的话会显示1而不是5
```



### 6.4 “==”判断

如果常量值在表达式右侧，代码将会编译和执行，并不像预期的那样。

如果常量值在表达式左侧，这个错误将在第一次尝试编译时被捕获。

应该：

```
if ( constant == var )
if ( NULL == pointer )
```

不应该：

```
if ( var == constant )
if ( pointer == NULL )
```

### 6.5 Getter 方法

`Getter` 方法全部写在文件所有方法之后，按照声明的顺序来写，用段注释和其他方法分隔：

```
#pragma mark - Getter
```

书写上建议统一使用下面这种方式，易复用。

应该：

```
- (UIButton *)loginButton {
    if (!_loginButton) {
        UIButton * button = [UIButton buttonWithType:UIButtonTypeCustom];
        [button setTitle:@"登录" forState:UIControlStateNormal];
        button.backgroundColor = [UIColor cyanColor];
        button.layer.cornerRadius = 4;
        button.clipsToBounds = YES;
        
        [self.view addSubview:button];
        _loginButton = button;
    }
    return _loginButton;
}
```

不应该：

```
- (UIButton *)loginButton {
    if (!_loginButton) {
        _loginButton = [UIButton buttonWithType:UIButtonTypeCustom];
        [_loginButton setTitle:@"登录" forState:UIControlStateNormal];
        _loginButton.backgroundColor = [UIColor cyanColor];
        _loginButton.layer.cornerRadius = 4;
        _loginButton.clipsToBounds = YES;
        [self.view addSubview:_loginButton];
    }
    return _loginButton;
}
```

### 6.6 属性特性的使用（copy/strong/retain）

`strong` 和 `retain` ：两者基本是一样的，在 ARC 环境下统一使用 strong。不一样的是 ARC 下的 strong 修饰 block 的时候会对 block 进行 copy 操作，不过还是建议保留好习惯，block 务必用 copy 修饰。

`copy` ：用于 NSString、NSArray、NSDictionary 等有对应的可变类型的属性，和 Block （具体原因见[官方文档：Objects Use Properties to Keep Track of Blocks](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithBlocks/WorkingwithBlocks.html#//apple_ref/doc/uid/TP40011210-CH8-SW12)）。

copy 特性：
    
> 1. 因为父类指针可以指向子类对象，使用 copy 的目的是为了让本对象的属性不受外界影响，使用 copy 无论给我传入是一个可变对象还是不可对象，我本身持有的就是一个不可变的副本。
  
> 2. 如果我们使用是 strong ，那么这个属性就有可能指向一个可变对象，如果这个可变对象在外部被修改了，那么会影响该属性。

### 6.7 属性特性的使用 （weak/assign）

`assign`：修饰基本数据类型和结构体。

`weak`：修饰指针变量。例如 delegate ，一定要用 weak 修饰。

 weak 比 assign 多了一个功能就是当属性所指向的对象消失的时候（也就是内存引用计数为0）会自动赋值为 nil ，这样再向 weak 修饰的属性发送消息就不会导致野指针操作 crash 。

### 6.8 Block 和 Delegate 的使用

Block 和 Delegate 回调之前要判断。

例如：

```
// Block
if (completionBlock) {
	completionBlock(operations); 
}

// Delegate
if ([delegate respondsToSelector:@selector(hudWasHidden:)]) {
	[delegate performSelector:@selector(hudWasHidden:) withObject:self];
}
```

### 6.9 .h 文件的书写

属性和方法都尽量写在 .m 文件中，不要轻易暴露，外部其他文件需要使用的时候才写在 .h 文件中。

### 6.10 扩展Category

文件命名格式如下：

> 类名 + 扩展名

```
UIImage+Extension
```

不要滥用扩展，需要扩展前检查下是否已有类似类，例如 UIImage 已有 UIImage+Extension ，包含了图片裁剪和方向调整，就不用另写 UIImage+Resize 扩展了，如果想添加方法，在原有扩展上加。

当然功能比较独立情况除外，例如 YYKit 中：

```
UIImage+YYAdd // 图片处理，例如裁剪、高斯模糊、圆角等
UIImageView+YYWebImage // 加载web图片
```

### 6.11   能用新的不用老的，比如YYKit

尽量使用新的技术。比如由于项目比较久比较庞大，以前都用 MJRefresh 的一些封装，后来有了 YYKit 对一些类的封装更好，新的代码使用 YYKit ，慢慢以旧代新。

## 七、工具推荐

### 7.1 Alcatraz

#### 7.1.1 Xcode8 兼容

Xcode8 开始苹果禁止使用插件，所以要做一次 XCode8 上的兼容，亲测有效：

- [兼容－记录Xcode8.0恢复插件全过程
](http://www.jianshu.com/p/738df17e9765)

> 安装 Alcatraz 时老是报错的话，直接从GitHub上下载 [ Alcatraz ](https://github.com/alcatraz/Alcatraz) 工程 -> 打开 Alcatraz 工程 -> 运行 Alcatraz 工程 -> 退出 Xcode -> 重新打开 Xcode -> `Load Bundle`。

#### 7.1.2 Xcode 中看不到 Package Manager

当时 OK 了，后来再打开 Xcode -> Window 又看不到 `Package Manager`  了，一番折腾后找到一篇文章，简单解决掉了。。。文章如下：

- [Xcode插件安装](http://www.cnblogs.com/calence/p/6060291.html)

#### 7.1.3 插件安装无效

终于能打开 Package Manager 方便的安装插件啦。但是安装后退出 Xcode ，再打开没有弹框是否 Load Bundles ，插件确实使用不了。解决如下：

在终端输入命令使用 `update_xcode_plugins` 更新插件后，再重新打开 Xcode 就会弹框是否 Load Bundles 了，选择 Load ，就可以使用了。

update_xcode_plugins 的安装和使用参考如下：

- [update_xcode_plugins](https://github.com/inket/update_xcode_plugins)

### 7.2 好用插件推荐
  
  以下是我用过觉得好用的插件推荐，大家还有什么好的插件的话欢迎推荐！

#### 7.2.1 FuzzyAutocomplete

[FuzzyAutocomplete](https://github.com/chendo/FuzzyAutocompletePlugin) 通过添加模糊匹配来提高 Xcode 代码自动补全功能，开发者无需遵循从头匹配的原则，只要记得方法里某个关键字即可进行匹配，很好地提高了工作效率。  

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/FuzzyAutocomplete.jpg)

#### 7.2.2  KSImageNamed

[KSImageNamed ](https://github.com/ksuther/KSImageNamed-Xcode)为项目中使用的 UIImage 的 imageNamed: 方法提供文件名自动补全功能。使用 [UIImage imageNamed:@"xxx"] 时，该插件会扫描整个 workspace 中的图片文件，显示图片信息。

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/KSImageNamed.jpg)

#### 7.2.3 OMColorSense

[OMColorSense](https://github.com/omz/ColorSense-for-Xcode) 。代码里的那些冷冰冰的颜色数值，到底时什么颜色？如果你经常遇到这个问题，每每不得不运行下代码去看看，那么这个插件绝对不容错过。更彪悍的是你甚至可以点击显示的颜色面板，直接通过系统的 ColorPicker 来自动生成对应颜色代码！

使用：任意一个颜色，点击颜色行，该行的右上角会出现色快，点击这个色块可以选择颜色。

#### 7.2.4 AutoHighlightSymbol

[AutoHighlightSymbol](https://github.com/chiahsien/AutoHighlightSymbol) 当你在 Xcode 选中一个词或者搜查一个词的时候标注高亮颜色，比 Xcode 自身的虚下划线明显得多。

#### 7.2.5 VVDocumenter

[VVDocumenter](https://github.com/onevcat/VVDocumenter-Xcode) 规范化代码注释，只需要输入三个斜线 `///`，就自动显示规范注释了。 

#### 7.2.6 FKRealGroup
[FKRealGroup](https://github.com/Forkong/FKRealGroup) 是一个增强Xcode创建、删除文件夹的插件。FKRealGroup会在编辑菜单中添加 `New Real Group` 和 `Delete Real Group` 两个选项。

Xcode本身的 `New Group` 选项只会创建一个虚拟文件夹，并不会在本地磁盘创建真实文件夹。`New Real Group` 选项会在相应磁盘目录创建一个真实的文件夹，`Delete Real Group` 同理。

#### 7.2.7 Auto-Importer

[Auto-Importer](https://github.com/citrusbyte/Auto-Importer-for-Xcode) 快速导入头文件。

使用方法：选中你要导入的头文件的名称，使用快捷键 `⌘ + ctrl + H` 即可。

#### 7.2.8 DXXcodeConsoleUnicodePlugin

[DXXcodeConsoleUnicodePlugin](https://github.com/dhcdht/DXXcodeConsoleUnicodePlugin) 转换 Xcode 控制台中  Unicode 为中文汉字。

使用方法（当然推荐使用方法2）
1. 快捷键 `option + c` 会转换当前剪贴板中的内容并用一个对话框把转换后的内容显示出来；

2. 在 `Xcode` 的 `Edit` 菜单中勾选 `ConvertUnicodeInConsole`，然后 console 中再出现 Unicode 码时，就会自动转换成与显示。

## 八、Xcode 快捷键和 Mac 快捷键

- [五分钟深度记忆你的Xcode快捷键](http://www.jianshu.com/p/419f2dddd58b)

- [Mac 键盘快捷键](https://support.apple.com/zh-cn/HT201236)

## 九、参考

- [**Apple**：Coding Guidelines for Cocoa](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)
- [**Google**：Google Objective-C Style Guide](https://google.github.io/styleguide/objcguide.xml)
- [**GitHub**：Style guide & coding conventions for Objective-C projects](https://github.com/github/objective-c-style-guide)
- [Raywenderlich官方英文版](https://github.com/raywenderlich/objective-c-style-guide)
- [Raywenderlich稍微改动翻译版](https://github.com/samlaudev/Objective-C-Coding-Style#error-handling)
- [Objective-C-Coding-Guidelines-In-Chinese](https://github.com/QianKaiLu/Objective-C-Coding-Guidelines-In-Chinese)

