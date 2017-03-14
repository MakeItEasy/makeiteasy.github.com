---
layout: post
title: "数字签名和数字证书的学习总结"
date: "2017-03-13 18:21:30"
comments: true
tags: [iOS]
---

本文主要是对网上资源的一些列举和总结，主要涉及的内容有以下几点：

* 非对称加密，对称加密，摘要算法
* 数字签名
* 数字证书
* SSL/TLS
* iOS中的证书以及签名过程

<!-- more -->

#### 非对称加密，对称加密，摘要算法

##### 非对称加密

参考[百度百科-非对称加密算法](http://baike.baidu.com/item/%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95)，总结如下：

* 需要一对公钥和私钥，对于一个私钥，只有一个对应的公钥
* 公钥可以公开，但是通过公钥几乎不可能推算出私钥
* 通过公钥加密的内容只有私钥才可以解密，反之，通过私钥加密的内容，也只有公钥才可以解密
* 缺点：加密速度慢，效率低

主要算法有：[RSA](http://baike.baidu.com/item/RSA%E7%AE%97%E6%B3%95)，[Diffie-Hellman算法](http://baike.baidu.com/item/Diffie-Hellman)

##### 对称加密

参考[百度百科-对称加密算法](http://baike.baidu.com/item/%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95)，总结如下：

* 双方使用同一个密钥进行加密和解密
* 加密速度快，加密效率高

主要算法有：[DES算法](http://baike.baidu.com/item/des%E7%AE%97%E6%B3%95)等


##### 摘要算法

参考[百度百科-摘要算法](http://baike.baidu.com/item/%E6%91%98%E8%A6%81%E7%AE%97%E6%B3%95)，总结如下：

* 具有不可逆性，也就是通过加密后的内容无法知道原文（否则能量不守恒~~）
* 不同的原文加密后的内容肯定不同

主要算法有：MD5，SHA1，SHA256

题外话，Google在2017年2月23日宣布攻破SHA-1加密技术，具体可参考[Google震惊密码界：攻破SHA-1加密技术](http://www.cnblogs.com/merlindu/p/6545588.html)，
但是也是非常费电费时的，不过保险起见，还是使用SHA256等更加不容易破解的加密方法。

#### 数字签名和数字证书

关于数字签名和数字证书，阮一峰老师的[这篇文章](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)描写的非常清晰。
我自己有如下一些认知：

数字签名就相当于是用自己的私钥对发布的内容进行加密后得到的密文（此处一般为了提高效率，一般会先对原文进行摘要加密，然后在对密文进行签名）

数字证书是为了解决客户端的公钥有可能被篡改，从而伪造中间人的问题。通过第三方CA认证机构签发证书，从而保证证书中的发布者的公钥是正确的。
数字证书会包含以下内容：

* 申请者的信息
* 申请者的公钥（很重要的信息）
* 使用证书发行方（CA机构）的私钥对整个证书信息进行加密后的签名（这个签名主要就是用来验证证书的内容是否被人篡改）
* 其他信息，比如证书有效期，CA机构的公钥等等

#### SSL/TLS

SSL/TLS的具体流程可参考阮一峰老师的文章：[SSL/TLS协议运行机制的概述](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html) 和 [图解SSL/TLS协议](http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html)，我的一些总结笔记如下：

* 生成对话密钥一共需要三个随机数，其中两个是客户端生成，一个是服务器生成，前两个随机数都是明文传递，只有第三个随机数是加密传递
* 整个握手阶段都不加密（也没法加密），都是明文的
* 握手阶段结束之后整个通话使用 `对话密钥` 进行对称加密
* 公钥私钥（非对称加密）只用于加密解密 `Premaster secret` (第三个随机数)，从而生成 `对话密钥`
* 服务器的公钥存放在服务器的数字证书中

#### iOS中签名，证书总结

关于iOS签名相关的内容，[漫谈iOS程序的证书和签名机制](https://segmentfault.com/a/1190000004144556)中的图和描述很好，可供参考。
以下是我的一些总结内容。

##### (一)苹果生成证书具体是干了什么？

1. 开发者在本地做成CertificateSigningRequest文件，其实就是生成公钥私钥，私钥保存在本地，公钥会在csr文件中，用于上传。
2. 开发者通过apple develop网站上传csr文件，生成证书。生成证书的过程其实就是苹果对开发者的信息（包括基本信息，公钥信息）进行哈希加密，然后使用自己的WWDR私钥对哈希加密的结果再次进行加密形成签名，同时将基本信息明文和签名结果一并合成到证书中。

##### (二)苹果系统如何验证证书？

1. 系统会拿到证书中的明文信息，然后进行哈希加密，得到哈希结果A
2. 系统使用WWDR公钥对证书中的签名信息进行解密，得到摘要信息B
3. 如果A和B相同的话，那么说明证书没有被篡改，从而可以拿出证书中的开发者的公钥，用于验证代码签名

##### (三)代码签名在干什么？

代码签名就是使用开发者的私钥对代码中的每个文件进行签名，然后把签名的结果放到一个文件中`_CodeSignature/CodeResources`

##### (四)iOS系统如何验证程序包？

1. 首先iOS会先提取出app包中的描述文件(mobileprovision文件)，该文件是通过apple develop网站生成下载的。
2. 确认描述文件没有被篡改过（方法同上面的(二)）
3. 如果描述文件正常，那么拿出描述文件中的证书信息，从证书中可以拿到开发者的公钥
4. 使用开发者的公钥对`_CodeSignature/CodeResources`中的签名进行解密，同时对指定的文件进行摘要加密，比对双方的结果

#### 参考链接博客中提出的一些命令总结

列出系统中可用于签名的有效证书：

```shell
security find-identity -v -p codesigning
```

查看苹果生成的证书中具体有哪些内容：

```shell
openssl x509 -inform der -in ios_development.cer -noout -text
```

查看CertificateSigningRequest文件中有啥内容：

```
openssl asn1parse -i -in CertificateSigningRequest.certSigningRequest
```

查看描述文件内容：

```
security cms -D -i embedded.mobileprovision
```

将描述文件中的DeveloperCertificates中的证书单独copy到一个文件中，组织成如下格式后，

```
# 这里要注意每行是64个字符，然后要有换行
-----BEGIN CERTIFICATE-----
MIIFnjCCBIagAwIBAgIIE/IgVItTuH4wDQYJKoZIhvcNAQEFBQAwgZYxCzA…
-----END CERTIFICATE-----
```

可通过命令查看：

```
openssl x509 -text -in file.pem
```

列出一些有关 Example.app 的签名信息:

```
codesign -vv -d Example.app
```

验证签名是否完好:

```
codesign --verify Example.app
```

#### 参考链接

* [百度文库-DH算法文档,里面有示例说明](http://wenku.baidu.com/view/a880a99f51e79b89680226f9.html)
* [数字签名是什么--阮一峰](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)
* [漫谈iOS程序的证书和签名机制](https://segmentfault.com/a/1190000004144556)
* [代码签名探析](https://objccn.io/issue-17-2/)
* [iOS Code Signing 学习笔记](http://foggry.com/blog/2014/10/16/ios-code-signing-xue-xi-bi-ji/)

(The End)
