---
title: "以太坊P2P发现机制之UDP握手(三)"
layout: post
date: 2018-09-03 23:00
tag: 
- 区块链
- P2P
- 发现机制
image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1542525464833&di=418b6d4878a23db3a42ecc6e68718b0c&imgtype=0&src=http%3A%2F%2Fpic2.zhimg.com%2Fv2-1cb5c8c5d171a5545fe2159b2fa3b21a_1200x500.jpg
headerImage: true
hidden: false # don't count this post in blog pagination
description: "以太坊P2P发现机制之UDP握手"
category: blockchain
author: augustus
externalLink: false
---

## 一、UDP握手协议产生的原因思考：

在TCP 建立连接、断开连接时，我们都知道是采用标准的三次握手和四次挥手协议。这样才保证了TCP连接的可靠。
在以太坊中，查询邻居节点，最近节点的请求采用的时UDP方式来执行的。那么为什么以太坊中需要对UDP专门建立相应的握手协议呢？？？思考了好久，我觉得如下理解应该是可以说的通的......

我的理解是：在P2P网络中，节点间都是可以互相发起访问的。如果A节点能请求B节点的数据，虽然对于A节点来说B节点是可通信的；但对于B节点来说，A节点不一定可以通信。举个例子：加入A节点处于环境NAT1中，而B节点则有独立的公网IP，此时A节点确实可以访问到B节点，而B节点则不一定能访问到A节点，因为A节点隐藏在了子网中，与其他子节点公用同一公网IP(这里可以参考UDP打洞相关知识)。

故而，我们在建立通信时，需要采用一种简单的握手协议，保证双方能完全的相互通信。


### 二、以太坊中UDP握手协议

理解了上述的原因后，我觉得就可以比较好的理解以太坊中实现UDP握手协议的流程了。从源码中可以构建出如下流程图：


<img class="image" src="{{ site.url }}/assets/images/cipher/udpws.png" alt="">
<figcaption class="caption">augustus</figcaption>


__对与 nodeA:__
     当其从db加载完所有的种子节点后，首先将所有的节点标记为 verifyinit(节点初始化状态)，然后对这批节点发起ping 请求，将已经发送完ping请求的节点
标记为 remoteverifywait(等待远程节点确认状态)。当在该状态收到远程节点的Pong包后，首先讲Pong包中的数据更新到本地(不变更状态)。 之后，如果收到了
远程节点的Ping 命令后，对A节点再次发送一个Pong命令作为回应，发送完Pong数据包后，更新A节点本地B节点的状态为Known状态，以此来标识握手完成。

__对于B节点__：
     当其受到A节点的Ping命令时，如果此时B节点内存中没有A节点的信息，那么A节点就是Unknown状态。受到节点A的Ping数据包后，B节点对Ping回应一个
Pong命令。发送完成后，主动发起一个对B的Ping数据包。完成这两步后 将A本地B节点的状态更新为verifywait状态。  之后当A节点收到其Pong回应后，将本地
节点A的状态更新为known。 以此来标识 A与B节点的握手完成。

当上述A\B节点都完成以上步骤，代表A,B两个节点都可以完全通信。这样就可以传输数据了。
