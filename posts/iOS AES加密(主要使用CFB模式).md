# iOS AES加密(主要使用CFB模式)

[toc]

## 前言
首先，上面是基本介绍，后面是个人经历的问题及解决方案，虽然写在最后，但是希望你们耐心看，不要犯我经历过的错误。祝你们都能顺利解决所有疑难杂症~

## 1. 介绍
### 1.1 AES是什么？

**[高级加密标准](https://zh.wikipedia.org/wiki/%E9%AB%98%E7%BA%A7%E5%8A%A0%E5%AF%86%E6%A0%87%E5%87%86)**（英语：Advanced Encryption Standard，缩写：AES），在密码学中又称 Rijndael 加密法。AES 是一个对称分组密码算法，旨在取代 DES 成为广泛使用的标准。

### 1.2  AES详解

AES 根据使用密码长度有三种方案以应对不同的场景要求，分别是 `AES-128`、`AES-192` 和 `AES-256`。加密模式有四种，分别是 `ECB`(Elecyronic Code Book,电子密码本)、`CBC`(Cipher Block Chaining,加密块链)、`CFB`(Cipher FeedBack Mode,加密反馈)、`OFB`(Output FeedBack,输出反馈)。需要和后台统一四个东西：`秘钥长度`、`加密模式`、`填充方式`、`初始向量`(也称偏移量，ECB模式不需要)。AES 加密原理如图：
![](https://github.com/LuckyCat7848/Blogs/blob/master/source/AESStructure.png)

**需要注意点：**
	1、iOS只支持 `PKCS7Padding` 填充方式，Java 支持 `PKCS5Padding` 但不支持PKCS7Padding，不过不要担心，在AES中这两个是相同的，具体不同之处自行百度；
	2、node.js 在AES加密上和其他语言有略不同，它系统自带方法对 Key 进行过 MD5 处理。

参考：[AES加密，Java、Node.js、Android、iOS跨平台使用](http://www.jianshu.com/p/fc524f8e85f3)


### 1.3 比较
- [AES五种加密模式（CBC、ECB、CTR、OCF、CFB）](http://www.cnblogs.com/baihuitestsoftware/articles/7090134.html)

### 1.4 实现原理
- [高级加密标准AES的工作模式（ECB、CBC、CFB、OFB）](https://blog.poxiao.me/p/advanced-encryption-standard-and-block-cipher-mode/)

### 1.5 模式和填充选择
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

## 2. 经验教训总结

### 2.1 加密模式和填充方式的确定

首先，**一定要确认使用的加密模式和填充方式！！！**因为我接到任务的时候邮件里没有说清楚，而 iOS 默认的是 CBC 模式，Android 是 CFB 模式一下就 OK 了，我调了半天不行，看了 Android 的代码才醒悟过来。

我的项目中，后台 使用的是`AES/CFB/PKCS7Padding`，Android 使用的是`AES/CFB/PKCS5Padding`。


### 2.2 填充方式的选择

其次，**填充方式的选择**，上面说到 `PKCS7Padding` 和 `PKCS5Padding`两种填充方式在 AES 下是一致的。

但是我的使用中，`CFB` 模式下加密和解密的结果并不一样！但是`CBC`模式下貌似可以，因为网上很多都是用 CBC 模式。

iOS 除了 PKCS7Padding ，还有一个`ccNoPadding`填充方式，使用该方式勉强 ok，举例如下：

例子： 
> 原字符串：@"hello中国"
> 原数据 Data：<68656c6c 6fe4b8ad e59bbd>

> Java 的`PKCS5Padding`方式加密后的字符串：@"QReiy/Ddik50cXQ="

> iOS 的`ccNoPadding`对 Java 加密字符串解密后 data：<68656c6c 6fe4b8ad e59bbd05 05050505>
iOS 的`ccNoPadding`对加密字符串解密后 string：@"hello中国\x05\x05\x05\x05\x05"	（这里要注意解密后的字符串在控制台输出是没问题的，但是实际是有多余的，如图：）
![](https://github.com/LuckyCat7848/Blogs/blob/master/source/AESExample.png)

> iOS 的`ccNoPadding`加密后的字符串：QReiy/DRrn92WV8=

可以看出来，因为一个是`PKCS5Padding`，一个是`ccNoPadding`会有填充模式上导致的数据差异。相信看过上面几篇文章的应该明白了。

> 如果加密的填充方式也是用`ccNoPadding`，那么解出来的就不会有多余填充了。**也就是说应该三方都保持同样的`ccNoPadding`填充方式，我和 Android 测试过（注意 Android 那边应该是 `NoPadding`书写方式）**。

但是后台那边说多个地方都已经使用，没有互相测试。也是怪我这边没有早点弄清楚这个问题，没有统一好使用方式。希望大家一切顺利！

###2.3 选错填充方式的补救

对此，只能我这边采取措施和 Android 、后台保持一致了。采取的方式很简单，就是在解密后台给的数据的时，截掉多余的填充。在加密传输时，加密后，补充需要填充的数据。这里主要是对 data 的操作。

### 2.4 加密方法的实现

```
    CCCryptorStatus cryptStatus = CCCrypt(kCCEncrypt,
                                          kCCAlgorithmAES128,
                                          kCCOptionPKCS7Padding,
                                          key.bytes,
                                          key.length,
                                          iv.bytes,
                                          self.bytes,
                                          self.length,
                                          buffer,
                                          bufferSize,
                                          &encryptedSize);
```

因为网上很多资源 AES 加密都是用如上方式实现，大家可以看到，这里没有 CFB/CBC 模式的字眼，因为这种方法的实现是默认 CBC 模式下的加密，如果使用其他模式，例如 CFB 模式，要是用 `CCCryptorCreateWithMode` 方法，该方法中可以设置不同的模式。具体参数类型两个方法点击进去对比一下就知道了，一个提供了加密模式参数`CCMode`，一个没有。

```
CCCryptorStatus CCCrypt(
    CCOperation op,         /* kCCEncrypt, etc. */
    CCAlgorithm alg,        /* kCCAlgorithmAES128, etc. */
    CCOptions options,      /* kCCOptionPKCS7Padding, etc. */
    const void *key,
    size_t keyLength,
    const void *iv,         /* optional initialization vector */
    const void *dataIn,     /* optional per op and alg */
    size_t dataInLength,
    void *dataOut,          /* data RETURNED here */
    size_t dataOutAvailable,
    size_t *dataOutMoved)
```
```
CCCryptorStatus CCCryptorCreateWithMode(
    CCOperation 	op,				/* kCCEncrypt, kCCDecrypt */
    CCMode			mode,
    CCAlgorithm		alg,
    CCPadding		padding,		
    const void 		*iv,			/* optional initialization vector */
    const void 		*key,			/* raw key material */
    size_t 			keyLength,	
    const void 		*tweak,			/* raw tweak material */
    size_t 			tweakLength,	
    int				numRounds,		/* 0 == default */
    CCModeOptions 	options,
    CCCryptorRef	*cryptorRef)	/* RETURNED */
```

#### 2.4.1 NSData 扩展实现解密解密

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

implementation NSData (EHIExtension)

/** AES解密：CFB模式 */
- (NSData *)aes256ByCFBModeWithOperation:(CCOperation)operation key:(NSString *)keyStr iv:(NSString *)ivStr {
    NSData *originData = self;
    NSUInteger mode = 16;
    if (operation == kCCEncrypt) {
        // 加密:位数不够的补全
        originData = [self fullData:originData mode:mode];
    }
    
    const char *iv = [[ivStr dataUsingEncoding:NSUTF8StringEncoding] bytes];
    const char *key = [[keyStr dataUsingEncoding:NSUTF8StringEncoding] bytes];
    
    CCCryptorRef cryptor;
    
    CCCryptorCreateWithMode(operation, kCCModeCFB, kCCAlgorithmAES, ccNoPadding, iv, key, [keyStr length], NULL, 0, 0, 0, &cryptor);
    
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
        originData = [self deleteData:resultData mode:mode];
    }
    
    return resultData;
}

/** 加密:位数不够的补全 */
- (NSData *)fullData:(NSData *)originData mode:(NSUInteger)mode {
    NSMutableData *tmpData = [[NSMutableData alloc] initWithData:originData];
    NSUInteger remainder = tmpData.length % mode;
    if (remainder != 0) {
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
    return originData;
}

/** 解密:位数多的删除 */
- (NSData *)deleteData:(NSData *)originData mode:(NSUInteger)mode {
    NSMutableData *tmpData = [[NSMutableData alloc] initWithData:originData];
    Byte *bytes = (Byte *)tmpData.bytes;
    Byte lastNo = bytes[tmpData.length - 1];
    if (lastNo >= 1 && lastNo <= 16) {
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

@end
```

#### 2.4.2 NSString 扩展实现使用过程
```
NSString+EHIAES.h：
#import <Foundation/Foundation.h>

@interface NSString (EHIAES)

/** AES解密 */
- (NSDictionary *)aes256Decrypt;

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

/** AES加密：key */
static NSString * const kAESKey = @""; // 32位
/** AES加密：iv */
static NSString * const kAESIv = @""; // 16位

@implementation NSString (EHIAES)

/** AES解密 */
- (NSDictionary *)aes256Decrypt {
    // 1.URL Decode
    NSString *urlDecodeStr = [self stringByURLDecode];
    // 2.Base64 Decode
    NSData *base64DecodeData = [NSData dataWithBase64EncodedString:urlDecodeStr];
    // 3.Aes256 解密
    NSData *decodeData = [base64DecodeData aes256ByCFBModeWithOperation:kCCDecrypt key:kAESKey iv:kAESIv];
    NSString *decodeStr = [[NSString alloc] initWithData:decodeData encoding:NSUTF8StringEncoding];
    if ([NSString isNilOrEmpty:decodeStr]) {
        // 解密失败
        return nil;
    }
    // 去掉多余的字符串 因为解密有补全机制,所以后面会有多余字符
    NSRange endRange = [decodeStr rangeOfString:@"}"];
    if (endRange.location != NSNotFound) {
        decodeStr = [decodeStr substringToIndex:endRange.location + 1];
    }
    // 4.json转字典
    NSError *error;
    NSData *jsonData = [decodeStr dataUsingEncoding:NSUTF8StringEncoding];
    NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:jsonData options:0 error:&error];
    if (error) {
        return nil;
    }
    return dic;
}

/** AES加密 */
- (NSString *)aes256Encrypt {
    NSData *originData = [self dataUsingEncoding:NSUTF8StringEncoding];
    // 1.Aes256 加密
    NSData *encodeData = [originData aes256ByCFBModeWithOperation:kCCEncrypt key:kAESKey iv:kAESIv];
    // 2.Base64 Encode
    NSString *base64EncodeStr = [encodeData base64EncodedString];
    // 3.URL Encode
    NSString *urlEncodeStr = [base64EncodeStr stringByURLEncode];
    return urlEncodeStr;
}

@end
```
