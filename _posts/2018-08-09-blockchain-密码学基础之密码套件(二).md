---
title: "密码学基础之密码套件(二)"
layout: post
date: 2018-08-09 22:10
tag: 
- 密码学基础
image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1542525464833&di=418b6d4878a23db3a42ecc6e68718b0c&imgtype=0&src=http%3A%2F%2Fpic2.zhimg.com%2Fv2-1cb5c8c5d171a5545fe2159b2fa3b21a_1200x500.jpg
headerImage: true
hidden: false # don't count this post in blog pagination
description: "以太坊密码学原理"
category: blockchain
author: augustus
externalLink: false
---

# 一、什么是密码套件

密码套件 是一个网络协议的概念。其中主要包括身份认证、加密、消息认证(MAC)、秘钥交换的算法组成。
在整个网络的传输过程中，根据密码套件主要分如下几大类算法：
* 秘钥交换算法：比如ECDHE、RSA。主要用于客户端和服务端握手时如何进行身份验证。
* 消息认证算法：比如SHA1、SHA2、SHA3。主要用于消息摘要。
* 批量加密算法：比如AES, 主要用于加密信息流。
* 伪随机数算法：<span data-type="color" style="color:rgb(34, 34, 34)"><span data-type="background" style="background-color:rgb(255, 255, 255)">例如TLS 1.2的伪随机函数使用MAC算法的散列函数来创建一个</span></span>__主密钥__<span data-type="color" style="color:rgb(34, 34, 34)"><span data-type="background" style="background-color:rgb(255, 255, 255)">——连接双方共享的一个48字节的私钥。主密钥在创建会话密钥（例如创建MAC）时作为一个熵来源。</span></span>

---

# 二、在网络中都涉及哪些加密呢？

在网络中，一次消息的传输一般需要在如下4个阶段分别进行加密，才能保证消息安全、可靠的传输。

#### 握手/网络协商阶段：
在双方进行握手阶段，需要进行链接的协商。主要的加密算法包括RSA、DH、ECDH等
#### 身份认证阶段：
身份认证阶段，需要确定发送的消失来源。主要采用的加密方式包括RSA、DSA、ECDSA(ECC加密，DSA签名)等。
#### 消息加密阶段：
消息加密指对发送的信息流进行加密。主要采用的加密方式包括DES、RC4、AES等。
#### 消息身份认证阶段/防篡改阶段：
主要是保证消息在传输过程中确保没有被篡改过。主要的加密方式包括MD5、SHA1、SHA2、SHA3等。


---

*<span data-type="color" style="color:#096DD9">在以太坊的RPC整个网络加密过程中，以上四个阶段都涉及到加密。握手协商阶段采用了ECDH加密算法，身份认证阶段采用了Secp256K1(是ECDSA算法中的一种)算法。消息机密阶段采用了AES算法。防篡改阶段采用了Keccak(一种SHA3)算法。</span>*









参考文献：
[https://zh.wikipedia.org/wiki/%E5%AF%86%E7%A0%81%E5%A5%97%E4%BB%B6](https://zh.wikipedia.org/wiki/%E5%AF%86%E7%A0%81%E5%A5%97%E4%BB%B6)

