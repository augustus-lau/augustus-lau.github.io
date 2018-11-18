---
title: "密码学之以太坊椭圆曲线(四)"
layout: post
date: 2018-09-10 22:10
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

## 一、概念

__ECC__<span data-type="color" style="color:rgb(79, 79, 79)"><span data-type="background" style="background-color:rgb(255, 255, 255)">：Elliptic Curves Cryptography，椭圆曲线密码编码学。是一种根据椭圆上点倍积生成 公钥、私钥的算法。用于生成公私秘钥。</span></span>

__ECDSA__<span data-type="color" style="color:rgb(79, 79, 79)"><span data-type="background" style="background-color:rgb(255, 255, 255)">：用于数字签名,是一种数字签名算法。一种有效的数字签名使接收者有理由相信消息是由已知的发送者创建的，从而发送者不能否认已经发送了消息(身份验证和不可否认)，并且消息在运输过程中没有改变。ECDSA签名算法是ECC与DSA的结合，整个签名过程与DSA类似，所不一样的是签名中采取的算法为ECC，最后签名出来的值也是分为r,s。</span></span><span data-type="color" style="color:rgb(79, 79, 79)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong>主要用于身份认证和签名阶段</strong></span></span><span data-type="color" style="color:rgb(79, 79, 79)"><span data-type="background" style="background-color:rgb(255, 255, 255)">。</span></span>

__ECDH__<span data-type="color" style="color:rgb(79, 79, 79)"><span data-type="background" style="background-color:rgb(255, 255, 255)">：也是基于ECC算法的霍夫曼树秘钥，通过ECDH，双方可以在不共享任何秘密的前提下协商出一个共享秘密，并且是这种共享秘钥是为当前的通信暂时性的随机生成的，通信一旦中断秘钥就消失。</span></span><span data-type="color" style="color:rgb(79, 79, 79)"><span data-type="background" style="background-color:rgb(255, 255, 255)"><strong>主要用于握手磋商阶段。</strong></span></span>

__ECIES：__是一种集成加密方案,也可称为一种混合加密方案，它提供了对所选择的明文和选择的密码文本攻击的语义安全性。ECIES可以使用不同类型的函数：秘钥协商函数(KA)，秘钥推导函数(KDF)，对称加密方案(ENC)，哈希函数(HASH), H-MAC函数(MAC)。

所以，__ECC__是椭圆加密算法，主要讲述了按照公私钥怎么在椭圆上产生，并且不可逆。__ECDSA__则主要是采用ECC算法怎么来做签名，__ECDH__则是采用ECC算法怎么生成对称秘钥。以上三者都是对ECC加密算法的应用。而现实场景中，我们往往会采用混合加密(对称加密，非对称加密结合使用，签名技术等一起使用)。__ECIES__就是底层利用ECC算法提供的一套集成(混合)加密方案。其中包括了非对称加密，对称加密和签名的功能。


## 二、ECC算法

### （一）ECC定义：

ECC 是 <span data-type="color" style="color:rgb(26, 26, 26)"><span data-type="background" style="background-color:rgb(255, 255, 255)">Elliptic Curve Cryptography的简称。那么什么是椭圆加密曲线呢？</span></span>Wolfram MathWorld 给出了很标准的定义：
__一条椭圆曲线就是一组被 __<img class="image" src="{{ site.url }}/assets/images/cipher/ecc2.svg" alt="">__ 定义的且满足 __<img class="image" src="{{ site.url }}/assets/images/cipher/ecc3.svg" alt="">__的点集。__<span data-type="color" style="color:rgb(26, 26, 26)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>
<img class="image" src="{{ site.url }}/assets/images/cipher/ecc4.svg" alt="">这个限定条件是为了保证曲线不包含奇点(singularities). 

<div style="text-align: center;" ><img class="image" src="{{ site.url }}/assets/images/cipher/ecc5.svg" alt=""></div>


<span data-type="color" style="color:rgb(26, 26, 26)"><span data-type="background" style="background-color:rgb(255, 255, 255)"> </span></span>这个方程称为椭圆曲线的维尔斯特拉斯标准形式（*Weierstrass normal form*<span data-type="color" style="color:rgb(26, 26, 26)"><span data-type="background" style="background-color:rgb(255, 255, 255)">）也就是我们的椭圆加密曲线。</span></span>

所以，随着曲线参数a和b的不断变化，曲线也呈现出了不同的形状。比如:


<img class="image" src="{{ site.url }}/assets/images/cipher/ecc1.png" alt="">
<figcaption class="caption">augustus</figcaption>



---


### （二）ECC的理解

所有的非对称加密的基本原理基本都是基于一个公式 K = k\*G。其中K代表公钥，k代表私钥，G代表某一个选取的基点。非对称加密的算法 就是要保证 该公式 __不可进行逆运算(__也就是说G/K是无法计算的__)。__

ECC是如何计算出公私钥呢？网上很多我都没有看懂，这里我只按照我自己的理解来描述。

__我理解，ECC的核心思想就是：随机在ECC曲线上取一个点k(作为私钥)，然后选择曲线上的一个基点G，然后根据k\*G计算出我们的公钥K。并且保证公钥K也要在曲线上。__

那么k\*G怎么计算呢？如何计算k\*G才能保证最后的结果不可逆呢？这就是ECC算法要解决的。


首先，我们先随便选择一条ECC曲线，a = -3, b = 7 得到如下曲线：


<img class="image" src="{{ site.url }}/assets/images/cipher/ecc6.png" alt="">
<figcaption class="caption">augustus</figcaption>



在这个曲线上，我随机选取两个点，这两个点的乘法怎么算呢？我们可以简化下问题，乘法是都可以用加法表示的，比如2\*2 = 2+2，3\*5 = 5+5+5。 那么我们只要能在曲线上计算出加法，理论上就能算乘法。


---

#### ECC加法体系

曲线上两点的加法又怎么算呢？这里ECC为了保证不可逆性，在曲线上自定义了加法体系。现实中，1+1=2，2+2=4，但在ECC算法里，我们理解的这种加法体系不存在。

__ECC定义，在图形中随机找一条直线，与ECC曲线相交于三个点(也有可能是两个点)，比三点分别是P、Q、R。__
__那么P+Q+R = 0。其中0 不是坐标轴上的0点，而是ECC中的无穷远点。也就是说定义了无穷远点为0点。__


<img class="image" src="{{ site.url }}/assets/images/cipher/ecc7.png" alt="">
<figcaption class="caption">augustus</figcaption>


同样，我们就能得出 P+Q = -R。 由于R 与-R是关于X轴对称的，所以我们就能在曲线上找到其坐标，入上图。

以上就描述了ECC曲线的世界里是如何进行加法运算的。


---


#### ECC 乘法运算

上面我们已经定义好了，在ECC曲线里，可以计算加法，那么我们就可以根据加法来推到出ECC里计算乘法。

要想计算乘法，我们先引入如下图形：




<img class="image" src="{{ site.url }}/assets/images/cipher/ecc8.png" alt="">
<figcaption class="caption">augustus</figcaption>


从上图可看出，直线与曲线只有两个交点，也就是说 直线是曲线的切线。此时P,R 重合了。
也就是P = R, 根据上述ECC的加法体系，P+R+Q = 0, 就可以得出 P+R+Q = 2P+Q = 2R+Q=0

于是乎得到 2\*P = Q (是不是与我们非对称算法的公式 K = k\*G 越来越近了)。

于是我们得出一个结论，可以算乘法，不过只有在切点的时候才能算乘法，而且只能算2的乘法。

假若 2 可以变成任意个数进行想乘，那么就能代表在ECC曲线里可以进行乘法运算，那么ECC算法就能满足非对称加密算法的要求了。


---


#### ECC 点倍积，满足曲线中的乘法运算

如上描述，我们按照 自定义的 加法体系，是可以算2\*P=Q这种乘法的，那么我们是不是可以随机任何一个数的乘法都可以算呢？ 答案是肯定的。

选一个随机数 k, 那么k \* P等于多少呢？

我们知道在计算机的世界里，所有的都是二进制的，ECC既然能算2的乘法，那么我们可以将随机数k描        述成二进制然后计算。假若k = 151 = 10010111



<img class="image" src="{{ site.url }}/assets/images/cipher/ecc9.png" alt="">

__所以 k\*P = 151 \* P =__  <img class="image" src="{{ site.url }}/assets/images/cipher/ecc10.png" alt="">

由于2\*P = Q 所以 这样就计算出了k\*P。__这就是点倍积算法__。

于是乎，我们得出结论，在ECC的曲线世界里，任何的乘法都是可以计算的。


---


### （三）ECC总结

ECC算法采用自己定义的加法体系，然后推到出在ECC曲线上是如何计算乘法运算。
在现实中，ECC算法去计算公钥和私钥就是采用了这种乘法运算。首先我们需要选择一条ECC的曲线(也就是定义曲线的a,b值)，之后定义一个点G为 曲线的基点，也就是无穷远点(或者0点)。

之后我们在曲线上随机选取一个节点(也就是我们的私钥) ， 然后用该私钥 去 乘以 基点G，然后得出一个值 K，如果计算得出的K值不在曲线上，则重新计算，直到算出的K在曲线上为止，则就把K当做我们的公钥。


至于为什么这样计算 是不可逆的。这需要大量的推演，我也不了解。但是我觉得可以这样理解：

我们的手表上，一般都有时间刻度。现在如果把1990年01月01日0点0分0秒作为起始点，如果告诉你至起始点为止时间流逝了 整1年，那么我们是可以计算出现在的时间的，也就是能在手表上将时分秒指针应该指向00：00：00。但是反过来，我说现在手表上的时分秒指针指向了00：00：00,你能告诉我至起始点算过了有几年了么？





## 二、ECDH算法

### (一)、ECDH定义与定位

ECDH算法是ECC与DH算法的结合。其中公私钥是采用ECC算法计算而出。ECDH是在秘钥协商阶段，在双方不交换秘钥的前提下，算出的一个共享秘钥。由于双方的公私秘钥都是基于ECC算法的，所以在算ECDH共享秘钥是，要保证双方是基于同样的椭圆曲线的。由于算出的秘钥是相同的，所以ECDH是对称秘钥，具有ECC的高强度，同时又兼具秘钥短，速度快等特点，所以在协议中广泛应用。以太坊在信息流传输阶段采用的就是这种方式。


### (二)、ECDH原理

在介绍原理前，说明一下ECC是满足结合律和交换律的，也就是说A+B+C = A+C+B = (A+C)+B。

这里举例说明如何生成共享秘钥，也可以参考[ Alice And Bob](https://en.wikipedia.org/wiki/Alice_and_Bob) 的例子。

Alice 与Bob 要进行通信，双方前提都是基于 同一参数体系的ECC生成的 公钥和私钥。所以有ECC有共同的基点G。

__生成秘钥阶段：__
Alice 采用公钥算法  KA = ka \* G ，生成了公钥KA和私钥ka, 并公开公钥KA。
Bob   采用公钥算法 KB = kb \* G ，生成了公钥KB和私钥 kb, 并公开公钥KB。

__计算ECDH阶段：__
Alice 利用计算公式  Q = ka \* KB  计算出一个秘钥Q。
Bob   利用计算公式  Q' = kb \* KA 计算出一个秘钥Q'。

__验证：__
Q = ka * KB = ka \* kb \* G = ka \* G \* kb = KA \* kb = kb \* KA = Q'*
故 双方分别计算出的共享秘钥不需要进行公开就可采用Q进行加密。我们将Q称为共享秘钥。


---



<em style="font-size: 12px;">参考连接:</em>

[ECC参考](http://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/)

[ECC图形演示](https://cdn.rawgit.com/andreacorbellini/ecc/920b29a/interactive/reals-add.html)

[ECDH参考资料](http://andrea.corbellini.name/2015/05/30/elliptic-curve-cryptography-ecdh-and-ecdsa/)

[ECDH交互过程WIKI](https://en.wikipedia.org/wiki/Alice_and_Bob)

[加密算法区分](https://crypto.stackexchange.com/questions/12823/ecdsa-vs-ecies-vs-ecdh)

[ECIES集成方案介绍](https://pdfs.semanticscholar.org/9f5e/ec8cb6a8883498157e8e27723da52ae4c752.pdf)

[ECIES在线演示](https://asecuritysite.com/encryption/ecc3)
