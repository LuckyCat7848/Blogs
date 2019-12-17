# 急急急! 快从UIWebView安全更新到WKWebView(包括做好线上兼容)

## 一、背景

查找很多零散博客，头疼，根据本次升级经验，还是自己整理出全面的一篇~✌️

公司项目相当老，苹果已下最后通牒，2020年4月之前必须更新为 WKWebView 了，就有了这次小心翼翼的升级。

首先考虑两点：
> 前端：JS 代码保证旧版和新版都能用，也就调用客户端 UIWebView 和 WKWebView 的方法都行得通才行；
> 客户端：写法确实有些不一样。

首先，Demo 写的很全，两端代码都有，方便学习，可以[直接下载](https://github.com/LuckyCat7848/LCWebViewDemo)对着博客查看。

-----------

更新：终于做完了，比想象的要辛苦。方案没有问题，但还是有些经验可以总结的，见第四节的总结（筋疲力尽~~~~~~~）。

## 二、JS 端

这一块可以提供给前端小伙伴，毕竟需要他们帮忙协助本次升级嘛~

### 2.1 缩放：解决滑动问题

```
“minimum-scale=1.0, maximum-scale=1.0, user-scalable=no”
```

### 2.2 JS调用iOS，传参数
    
```  
try { // 新
    window.webkit.messageHandlers.callAMethod.postMessage('callcallcall');
} catch (error) { // 旧
    window.callAMethod('callcallcall');
}
```

> **注意：**
> 1. 新版postMessage必须有参数，无也要传参：`.postMessage(null）`；
> 2. 多个参数，打包传参：
> 数组：`.postMessage(['第一个参数'，'第二个参数'，'第三个参数'])`
> 字典：`.postMessage({'arg1' : '第一个参数', 'arg2' : '第二个参数', 'arg3' : '第三个参数'})`

### 2.3 JS调用iOS，并🌹同步的🌹拿到返回值

> **注意：先约定 type 为`JSbridge`**。

这里的try-catch是demo要这么写，前端同学有自己的判断代码的，不贴了哦。

```
var result;
try { // 新
    var type = "JSbridge";
    var functionName = "getReturnValue:";
    var args = "一个参数";

    var payload = {"type": type, "functionName": functionName, "arguments": args};
    result = prompt(JSON.stringify(payload));
    
} catch (error) { // 旧
    result = window.getReturnValue();
}
```

### 2.4 iOS主动调用JS，或作为JS🌹异步的🌹拿到传值

```
// 🌹异步的🌹响应客户端的调用
function jsResponseClient(response) {
    alert(response);
}
```

## 三、客户端

这块代码太多了... 上面 JS 那块代码及说明友友们需要粘走给前端小伙伴查看。这里咱们自己看，我就写重点了哈。

### 3.1 响应JS调用

见代码吧亲，注释也很详细，这里就提醒两点：

1. 和UIWebView不同的是，生成 webView 的时候就先注册和 JS 交互的方法名；

```
// 创建UserContentController（提供JavaScript向webView发送消息的方法）
WKUserContentController *userContent = [[WKUserContentController alloc] init];
// JS回调方法的监听,这里要注意避免释放问题
// (懒得引入YY或写一个了,写的时候别忘了哈)
id handler = self;//[YYWeakProxy proxyWithTarget:self];
for (NSString *name in self.scriptNameList) {
    [userContent addScriptMessageHandler:handler name:name];
}
```

2. 接收 JS 调用的代理方法。

```
webView.navigationDelegate = self;
```

```
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    
    if ([message.name isEqualToString:kJSMoreParameter]) {
        // 多个参数
        NSLog(@"%@", message.body);
    }
}
```

### 3.2 响应JS调用，并同步返回值

这里同步的给 JS 返回值比较特别，需要先提到 JS 的弹框 WKWebView 不显示问题，需要 webView 接受以下代理方法，并代码显示弹框：

```
webView.UIDelegate = self;
```

```
/** 显示一个按钮。点击后调用completionHandler回调 */
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler {
    
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:message message:nil preferredStyle:UIAlertControllerStyleAlert];
    [alertController addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        completionHandler();
    }]];
    [self presentViewController:alertController animated:YES completion:nil];
}

/** 显示两个按钮。通过completionHandler回调判断用户点击的确定还是取消按钮 */
- (void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL))completionHandler {
    
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:message message:nil preferredStyle:UIAlertControllerStyleAlert];
    [alertController addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        completionHandler(YES);
    }]];
    [alertController addAction:[UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        completionHandler(NO);
    }]];
    [self presentViewController:alertController animated:YES completion:nil];
}

/** 显示一个带有输入框和一个确定按钮的。通过completionHandler回调用户输入的内容 */
- (void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * _Nullable))completionHandler {
    
    // 拦截弹框,同步返回传值给JS
    NSError *err = nil;
    NSData *dataFromString = [prompt dataUsingEncoding:NSUTF8StringEncoding];
    NSDictionary *payload = [NSJSONSerialization JSONObjectWithData:dataFromString options:NSJSONReadingMutableContainers error:&err];
    if (!err) {
        NSString *type = [payload objectForKey:@"type"];
        if (type && [type isEqualToString:@"JSbridge"]) {
            NSString *returnValue = @"";
            NSString *functionName = [payload objectForKey:@"functionName"];
            NSDictionary *args = [payload objectForKey:@"arguments"];
            
            SEL selector = NSSelectorFromString(functionName);
            if ([self respondsToSelector:selector]) {
                returnValue = [self performSelector:selector withObject:args];
            }
            completionHandler(returnValue);
            return ;
        }
    }
    
    // 显示弹框
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"" message:nil preferredStyle:UIAlertControllerStyleAlert];
    [alertController addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
        
    }];
    [alertController addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        completionHandler(alertController.textFields.lastObject.text);
    }]];
    [self presentViewController:alertController animated:YES completion:nil];
}
```

呐，重点就是上面的`拦截弹框,同步返回传值给JS`啦~

原因就是 JS 同步拿 WKWebView 的返回值方法也比较特殊，是通过`promot`弹框方式拿的。所以我们把弹框拦截，同步给返回值即可。

### 3.3 主动调用JS，或作为异步返回值给JS

```
/** 主动调用JS方法 */
- (void)callJSMethod {
    NSString *aString = @"客户端主动调用JS方法，也可用于JS🌹异步的🌹拿到客户端返回值";
    NSString *func = [NSString stringWithFormat:@"jsResponseClient('%@')", aString];
    [self.webView evaluateJavaScript:func completionHandler:^(id _Nullable object, NSError * _Nullable error) {
        NSLog(@"%@   %@", object, error);
    }];
}
```

以上相关功能 UIWebView 也全部实现，代码就不贴了，见 Demo 哈。

-----------

更新：

## 四、总结

做完很辛苦，都是泪，然后还是有些经验可以总结的。。。

1. 首先，如果你的项目很老，写一堆太多了。我们要注重项目结构，使用分类来分担业务功能：

```

- EHIWebDefines // 事件名、静态URL等
- EHIWebViewController+Intercept // 发送请求前的拦截操作,例如h5里的调微信/支付宝支付
- EHIWebViewController+JSAction // JS调用的一堆事件
- EHIWebViewController+JSGetValue // JS调用同步获取值

```

2. JSAction 和 JSGetValue 中是方法实现，调起的优化可以使用 Runtime 优雅实现：

```

#pragma mark - WKScriptMessageHandler ⚠️JS调用（具体实现在JSAction中）

- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
    NSString *functionName = message.name;
    // 函数统一都加分号(避免后来再加参数,前面版本不兼容)
    if (functionName.length && ![functionName hasSuffix:@":"]) {
        functionName = [NSString stringWithFormat:@"%@:", functionName];
    }
    NSDictionary *args = message.body;
    
    SEL selector = NSSelectorFromString(functionName);
    if ([self respondsToSelector:selector]) {
        dispatch_async(dispatch_get_main_queue(), ^{
            kSuppressPerformSelectorLeakWarning([self performSelector:selector withObject:args]);
        });
    }
}

#pragma mark - WKUIDelegate ⚠️JS弹框的显示 或 同步获取值（具体实现在JSGetValue中）
/** 显示一个带有输入框和一个确定按钮的。通过completionHandler回调用户输入的内容 */
- (void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * _Nullable))completionHandler {
    
    // 拦截弹框,同步返回传值给JS
    NSError *err = nil;
    NSData *dataFromString = [prompt dataUsingEncoding:NSUTF8StringEncoding];
    NSDictionary *payload = [NSJSONSerialization JSONObjectWithData:dataFromString options:NSJSONReadingMutableContainers error:&err];
    if (!err) {
        NSString *type = [payload objectForKey:@"type"];
        if (type && [type isEqualToString:@"JSbridge"] && [webView.URL.host containsString:@"你的host,防止外者调用"]) {
            NSString *returnValue = @"";
            NSString *functionName = [payload objectForKey:@"functionName"];
            NSDictionary *args = [payload objectForKey:@"arguments"];
            
            SEL selector = NSSelectorFromString(functionName);
            if ([self respondsToSelector:selector]) {
                kSuppressPerformSelectorLeakWarning(returnValue = [self performSelector:selector withObject:args]);
            }
            completionHandler(returnValue);
            return ;
        }
    }
    
    // 显示弹框
    // ...
}

```

3. 需要注册很多个方法也很烦，虽然我们项目老不会一次弄好，但是思考一下，以后可以用得着：

只注册一个方法，参数为 URL，就可以使用中间件来自由调用了！从扩展和维护角度都更好啊！！！

不做过这么折腾的事情是体会不到的，真的太痛苦了！好好写代码，为后人少点坑，ending...😭
