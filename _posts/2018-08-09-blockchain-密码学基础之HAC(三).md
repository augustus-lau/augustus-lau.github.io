---
title: "密码学基础之HMAC(三)"
layout: post
date: 2018-08-09 23:01
tag: 
- 密码学基础
image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1542525464833&di=418b6d4878a23db3a42ecc6e68718b0c&imgtype=0&src=http%3A%2F%2Fpic2.zhimg.com%2Fv2-1cb5c8c5d171a5545fe2159b2fa3b21a_1200x500.jpg
headerImage: true
hidden: false # don't count this post in blog pagination
description: "以太坊密码学原理之HMAC"
category: blockchain
author: augustus
externalLink: false
---

# 一、H-MAC与比特币

[H-MAC](https://en.wikipedia.org/wiki/HMAC)全程叫做 Hash-based Message Authentication Code. 其流程如下：



<img class="image" src="{{ site.url }}/assets/images/cipher/hmac1.png" alt="">
<figcaption class="caption">augustus</figcaption>

---

     
在比特币交互里，SENDER要与RECEIVER交互，首先SENDER要想比特币中心申请一个私钥。而比特币中心也保留该私钥(对称加密)。
首先，SENDER要发送消息，先将消息经过 私钥 和 MAC算法  生成MAC
其次，SENDER将消息 + MAC 都发送个比特币服务器。
当比特币接收到消息后，采用保留的私钥 和Mac 算法 对message 计算得出MAC
最后，比较计算出的mac与接收到的mac。这样来以防篡改。

---


# 二、以太坊中UDP消息加密模型

## (一)、以太坊UDP请求体结构

在以太坊的请求体的中，包含如下内容：mac  , sig , ptype, message.
其中 mac 占 256位，sig 占520位，ptype 占8为， mac+sig 共称为header
mac：请求摘要
sig:    数字签名(私钥签名)
ptype: 事件类型
message:消息内容
模型如下：


<img class="image" src="{{ site.url }}/assets/images/cipher/hmac2.png" alt="">
<figcaption class="caption">augustus</figcaption>

---


## (二)、认证、签名流程

在以太坊UDP交互中，没有对数据进行加密，只是采用了[RLP](https://www.jianshu.com/p/da638d0fcea4)编码的明文来传输数据。

发送方：
1、对message 进行[rlp](https://www.jianshu.com/p/da638d0fcea4) 编码
2、对ptype + message编码后的数据  计算 hash算法(采用Keccak256算法)。
3、对2中的hash值 利用私钥 做数字签名 得到sig。
4、对 sig+ptype+message 进过hash算法 计算摘要mac.

接收方：
接收方接受到数据后，做逆向解析操作。


#### __[问题]__

对于上述流程，其实有一个问题，那就是经过发送方的私钥加密后，看起来是密文，但是A的公钥是公开的，意味着网络中其他节点都可以读取到该消息。也就是其他节点都可以解密。

#### __[理解解答]__
	
其实，第三步利用私钥签名，并不是要为了加密，而是只是为了签名，证明该消息是由 该节点发送的。
为了防止有人拦截该节点消息，并伪造该节点发送数据，这里对 sig+ptype+data 生成一次摘要。该摘要起到保护 签名 、类型、数据都不允许修改的目的。

## (三)、传输，加密，认证，签名流程模型

以太坊UDP间的数据传输流程如下：



<img class="image" src="{{ site.url }}/assets/images/cipher/hmac3.png" alt="">
<figcaption class="caption">augustus</figcaption>

---

这里着重说一下接收方：
首先，接收到数据，提取其中的mac信息摘要，
其次，对sig+ptype+rlp-data 计算keccak的hash算法，得出mac1.
认证：校验mac与mac1是否相同，如相同继续下面流程
__重点：接下来将 ptype+rlp-data  做 __[keccak](https://keccak.team/software.html)__的hash算法得到hash值。 根据hash值 和 sig 计算出公钥(Sender的公钥么)。__
最后根据 公钥 位移 转换出Sender的 NODEID.
然后，根据ptype类型不同，封装不同的接收方数据包，将rlp-data解码到数据包中。


