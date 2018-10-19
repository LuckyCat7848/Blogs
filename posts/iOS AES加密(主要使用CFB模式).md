# iOS AES加密(主要使用CFB模式)

## 前言

首先，希望大家耐心点，这个加密我也是弄很久才出来的，辛辛苦苦整理的博客，介绍大概概念就进入正题！

上面是基本介绍，**`后面是个人经历的问题及解决方案`**，希望你们耐心看，不要踩我进过的坑啦！祝你们都能顺利解决问题✌️

## 1. 介绍
### 1.1 AES是什么？

**[高级加密标准](https://zh.wikipedia.org/wiki/%E9%AB%98%E7%BA%A7%E5%8A%A0%E5%AF%86%E6%A0%87%E5%87%86)**（英语：Advanced Encryption Standard，缩写：AES），在密码学中又称 Rijndael 加密法。AES 是一个对称**`分组密码`**算法，旨在取代 DES 成为广泛使用的标准。

### 1.2  AES详解

AES 根据使用密码长度有三种方案以应对不同的场景要求，分别是 `AES-128`、`AES-192` 和 `AES-256`。加密模式有四种，分别是 `ECB`(Elecyronic Code Book,电子密码本)、`CBC`(Cipher Block Chaining,加密块链)、`CFB`(Cipher FeedBack Mode,加密反馈)、`OFB`(Output FeedBack,输出反馈)。

> **需要和后台统一四个东西：`秘钥长度`、`加密模式`、`填充方式`、`初始向量`(也称偏移量，ECB模式不需要)。**

定义中说到 AES 是一个对称**分组密码**算法，加密原理如图：

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/AESStructure.png)

### 1.3 实现原理和比较

这个就比较深入啦，有兴趣的自行查看~

1. 实现原理
- [高级加密标准AES的工作模式（ECB、CBC、CFB、OFB）](https://blog.poxiao.me/p/advanced-encryption-standard-and-block-cipher-mode/)

2. 比较
- [AES五种加密模式（CBC、ECB、CTR、OCF、CFB）](http://www.cnblogs.com/baihuitestsoftware/articles/7090134.html)

### 1.4 模式和填充选择

- [真正跨平台的AES加密/解密方案. 支持 Java,C,nodeJs,Android,IOS...](https://github.com/keel/aes-cross/tree/master/info-cn)

算法/模式/填充            |16字节加密后数据长度|不满16字节加密后长度
-------------------------|---------------|-------------------
AES/CBC/NoPadding        |     16        |   不支持
AES/CBC/PKCS5Padding     |     32        |   16
AES/CBC/ISO10126Padding  |     32        |   16
AES/CFB/NoPadding        |     16        |   原始数据长度
AES/CFB/PKCS5Padding     |     32        |   16
AES/CFB/ISO10126Padding  |     32        |   16
AES/ECB/NoPadding        |     16        |   不支持
AES/ECB/PKCS5Padding     |     32        |   16
AES/ECB/ISO10126Padding  |     32        |   16
AES/OFB/NoPadding        |     16        |   原始数据长度
AES/OFB/PKCS5Padding     |     32        |   16
AES/OFB/ISO10126Padding  |     32        |   16
AES/PCBC/NoPadding       |     16        |   不支持
AES/PCBC/PKCS5Padding    |     32        |   16
AES/PCBC/ISO10126Padding |     32        |   16

**PKCS7Padding VS PKCS5Padding**：
`PKCS5Padding` 的 blocksize 为8字节，而 `PKCS7Padding` 的 blocksize 可以为1到255字节。

> **需要注意点：**
	1. iOS只支持 `PKCS7Padding` 填充方式；Java 支持 `PKCS5Padding` 但不支持 `PKCS7Padding`，不过不要担心，上面说的区别我也不懂，实际中倒是一样；
	2. node.js 在 AES 加密上和其他语言有略不同，它系统自带方法对 Key 进行过 MD5 处理。

## 2. 经验总结

### 2.1 加密模式和填充方式的确定

首先，**一定要确认使用的加密模式和填充方式！！！**因为我接到任务的时候邮件里只给了 `key` 和 `iv`，没有说清楚，而 `iOS` 默认的是 `CBC` 模式，`Android` 是 `CFB` 模式一下就 OK 了，我调了半天不行，看了 `Android` 的代码才醒悟过来。。。。。。

> **我的项目中，后台使用的是`AES/CFB/PKCS7Padding`，Android 使用的是`AES/CFB/PKCS5Padding`。**


### 2.2 填充方式的选择

其次，**填充方式的选择**：按照上面来看，我使用 `AES/CFB/PKCS7Padding` 就可以了哈。
然而，iOS 有的加密方法，只有 `CCCryptorCreateWithMode` 可以设置除了默认的 CBC 、ECB 之外的其他模式，所以就用它啦，其方法如下：
```
CCCryptorStatus status = CCCryptorCreateWithMode(operation,
                                                     kCCModeCFB,
                                                     kCCAlgorithmAES,
                                                     ccPKCS7Padding,
                                                     iv,
                                                     key,
                                                     keyStr.length,
                                                     NULL,
                                                     0,
                                                     0,
                                                     0,
                                                     &cryptor); 
```
这里的 `padding` 除了 `ccPKCS7Padding`，还有 `ccNoPadding` 不填充两种选择。我试了两个的加密结果是一样的，使用 `ccPKCS7Padding` 并没有自动填充。（可能其他模式可以，总不能有这个还不能用吧。但是我没有查到 `AES/CFB/PKCS7Padding` 为什么不填充，如果小伙伴知道还请告知哦！）

> **所以我就先试试 `ccNoPadding` 不填充模式。**举例如下：

原字符串：@"hello中国"
原数据 Data：<68656c6c 6fe4b8ad e59bbd>

Java 的`PKCS5Padding`方式加密后的字符串：QReiy/Ddik50cXQ=
iOS 的`ccNoPadding`加密后的字符串：QReiy/DRrn92WV8= 

加密结果当然不一致，下面对 Java 加密字符串解密后进行分析：

> iOS 的`ccNoPadding`对 Java 加密字符串解密后 data：<68656c6c 6fe4b8ad e59bbd05 05050505>
iOS 的`ccNoPadding`对加密字符串解密后 string：@"hello中国\x05\x05\x05\x05\x05"（这里要注意解密后的字符串在控制台输出是没问题的，但是实际是有多余的，如图：）

![](https://github.com/LuckyCat7848/Blogs/blob/master/source/AESExample.png)

可以看出来，因为一个是`PKCS5Padding`，一个是`ccNoPadding` 会有填充模式上导致的数据差异。相信看过上面几篇文章的应该明白了。

> 如果 Java 加密的填充方式也是用`ccNoPadding`，那么解出来的就不会有多余填充了。**也就是说应该三方都保持同样的`ccNoPadding`填充方式，我和 Android 测试过（注意 Android 那边应该是 `NoPadding`书写方式）**。

但是后台那边说多个地方都已经使用 `PKCS7Padding`。哎，也是怪我这边没有早点弄清楚这个问题，没有统一好使用方式。希望大家一切顺利！当然解决办法还是有的~~~

###2.3 选错填充方式的补救

对此，只能我这边采取措施和 Android 、后台保持一致了。
也就是**解密后台给的数据的时，截掉多余的填充；加密传输时，加密后，补充需要填充的数据。**这里主要是对 `NSData` 的操作。

**注意：**加解密的步骤（ase64、URL Encode、有的还有字符串替换）不同公司可能采取方式不同，要对接好。

### 2.4 代码思路

- [iOS 实现对称加密多种填充方式(ANSIX923、ISO10126、Zero)](https://www.jianshu.com/p/7b6f5aaa7680)

在我的项目中，对 NSData 进行填充补位和删除，我们需要了解 `PKCS7Padding` 的填充方式：

>  **`需要填充的字节长度 = (块长度 - (数据长度 % 块长度))`**

```
假定块长度为 8，数据长度为 3，则填充字节数等于 5。
原数据为： FF FF FF 
填充结果： FF FF FF 05 05 05 05 05 
```
```
假定块长度为 8，数据长度为 8，则填充字节数等于 8。
原数据为： FF FF FF FF FF FF FF FF
填充结果： FF FF FF FF FF FF FF FF 08 08 08 08 08 08 08 08 
```
差多少补多少，不差就补一个块。

> 当然我采坑的过程中没那么顺利 ，在此想提醒大家这种不熟悉的任务一定要**多批量复杂数据**，不能简简单单测试简单少量数据就行了，早发现早解决。

### 2.5 加密方法的实现

#### 2.5.1 NSData 扩展实现加密解密

NSData+EHIExtension.h：
```
#import <Foundation/Foundation.h>
#include <CommonCrypto/CommonCrypto.h>

interface NSData (EHIExtension)

/** AES解密：CFB模式 */
- (NSData *)aes256ByCFBModeWithOperation:(CCOperation)operation key:(NSString *)keyStr iv:(NSString *)ivStr;

end
```
NSData+EHIExtension.m：
```
#import "NSData+EHIExtension.h"

/** AES加密位数 */
static NSInteger const kEHIAESMode = 16;

@implementation NSData (EHIExtension)

/** AES解密：CFB模式 */
- (NSData *)aes256ByCFBModeWithOperation:(CCOperation)operation key:(NSString *)keyStr iv:(NSString *)ivStr {
    NSData *originData = self;
    if (operation == kCCEncrypt) {
        // 加密:位数不够的补全
        originData = [self fullData:originData mode:kEHIAESMode];
    }
    
    const char *iv = [[ivStr dataUsingEncoding:NSUTF8StringEncoding] bytes];
    const char *key = [[keyStr dataUsingEncoding:NSUTF8StringEncoding] bytes];
    
    // 加密/解密
    CCCryptorRef cryptor = NULL;
    CCCryptorStatus status = CCCryptorCreateWithMode(operation,
                                                     kCCModeCFB,
                                                     kCCAlgorithmAES,
                                                     ccNoPadding,
                                                     iv,
                                                     key,
                                                     keyStr.length,
                                                     NULL,
                                                     0,
                                                     0,
                                                     0,
                                                     &cryptor);
    if (status != kCCSuccess) {
        NSLog(@"AES加密/解密失败 error: %@", @(status));
        return nil;
    }
    
    // 输出加密/解密数据
    NSUInteger inputLength = originData.length;
    char *outData = malloc(inputLength);
    memset(outData, 0, inputLength);
    
    size_t outLength = 0;
    CCCryptorUpdate(cryptor, originData.bytes, inputLength, outData, inputLength, &outLength);
    NSData *resultData = [NSData dataWithBytes:outData length:outLength];
    
    CCCryptorRelease(cryptor);
    free(outData);
    
    if (operation == kCCDecrypt) {
        // 解密:位数多的删除
        resultData = [self deleteData:resultData mode:kEHIAESMode];
    }
    return resultData;
}

/** 加密:位数不够的补全
    补位规则：1.length=13,补5位05
            2.length=16,补16位ff */
- (NSData *)fullData:(NSData *)originData mode:(NSUInteger)mode {
    NSMutableData *tmpData = [[NSMutableData alloc] initWithData:originData];
    // 确定要补全的个数
    NSUInteger shouldLength = mode * ((tmpData.length / mode) + 1);
    NSUInteger diffLength = shouldLength - tmpData.length;
    uint8_t *bytes = malloc(sizeof(*bytes) * diffLength);
    for (NSUInteger i = 0; i < diffLength; i++) {
        // 补全缺失的部分
        bytes[i] = diffLength;
    }
    [tmpData appendBytes:bytes length:diffLength];
    return tmpData;
}

/** 解密:位数多的删除
    删位规则：最后一位数字在1-16之间,且连续n位相同n数字 */
- (NSData *)deleteData:(NSData *)originData mode:(NSUInteger)mode {
    NSMutableData *tmpData = [[NSMutableData alloc] initWithData:originData];
    Byte *bytes = (Byte *)tmpData.bytes;
    Byte lastNo = bytes[tmpData.length - 1];
    if (lastNo >= 1 && lastNo <= mode) {
        NSUInteger count = 0;
        // 确定多余的部分正确性
        for (NSUInteger i = tmpData.length - lastNo; i < tmpData.length; i++) {
            if (lastNo == bytes[i]) {
                count ++;
            }
        }
        if (count == lastNo) {
            // 截取正常的部分
            NSRange replaceRange = NSMakeRange(0, tmpData.length - lastNo);
            return [tmpData subdataWithRange:replaceRange];
        }
    }
    return originData;
}
```

#### 2.5.2 NSString 扩展实现使用过程

NSString+EHIAES.h：
```
#import <Foundation/Foundation.h>

interface NSString (EHIAES)

/** AES解密 */
- (NSString *)aes256Decrypt;

/** AES加密 */
- (NSString *)aes256Encrypt;

@end
```
NSString+EHIAES.m：
```
#import "NSString+EHIAES.h"
#import "NSString+YYAdd.h"
#import "NSData+YYAdd.h"
#import "NSData+EHIExtension.h"
#import "GTMBase64.h"

/** AES加密：key */
static NSString * const kAESKey = @""; // 32位
/** AES加密：iv */
static NSString * const kAESIv = @""; // 16位

@implementation NSString (EHIAES)

/** AES解密 */
- (NSString *)ehi_aes256Decrypt {
    // 1.Base64 Decode
    NSData *base64DecodeData = [NSData dataWithBase64EncodedString:self];
    // 1.Aes256 解密
    NSData *decodeData = [base64DecodeData aes256ByCFBModeWithOperation:kCCDecrypt key:kAESKey iv:kAESIv];
    NSString *decodeStr = [[NSString alloc] initWithData:decodeData encoding:NSUTF8StringEncoding];
    if ([NSString isNilOrEmpty:decodeStr]) {
        // 解密失败
        return nil;
    }
    return decodeStr;
}

/** AES加密 */
- (NSString *)ehi_aes256Encrypt {
    NSData *originData = [self dataUsingEncoding:NSUTF8StringEncoding];
    // 1.Aes256 加密
    NSData *encodeData = [originData aes256ByCFBModeWithOperation:kCCEncrypt key:kAESKey iv:kAESIv];
    // 2.Base64 Encode
    NSString *base64EncodeStr = [encodeData base64EncodedString];
    return base64EncodeStr;
}

@end
```
