---
title: "以太坊P2P发现机制(一)"
layout: post
date: 2018-09-01 23:00
tag: 
- 区块链
- P2P
- 发现机制
image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1542525464833&di=418b6d4878a23db3a42ecc6e68718b0c&imgtype=0&src=http%3A%2F%2Fpic2.zhimg.com%2Fv2-1cb5c8c5d171a5545fe2159b2fa3b21a_1200x500.jpg
headerImage: true
hidden: false # don't count this post in blog pagination
description: "以太坊P2P发现机制"
category: blockchain
author: augustus
externalLink: false
---

## 一、路由表在P2P中的作用与简介

在P2P网络环境中，由于资源是分布式存放的，当我们查找某一个资源时，如何能快速定位到资源所在呢？这其实就是路由表的作用所在，我们常用的迅雷，电驴也都是采用了路由表的方式快速锁定资源去下载。

那么路由表的实现原理是什么呢？

我举例来说明下，然后再引入KAD算法，以方便说明该算法能解决什么问题：
假如有个P2P集群，其中每个节点都保存了其他节点的信息，并且每个节点都有唯一的一个ID。但有一条数据(已知该数据的KEY)要添加到集群中时，具体该怎么添加呢？

随便添加在某台机器上肯定不行，以后都没有办法查询；
我们对资源KEY 和 节点的ID用同种HASH算法计算，然后将资源存在相同HASH的节点上。这种方式也不合适；因为__一是 不能保证一定能找到与资源KEY的HASH值相同的节点__；__二是 即使找到了，集群动态扩容时数据迁移量会很大，如不迁移数据就无法被查询到__。__三是 如果某个节点失联，其上的数据也将无法被查询到。__

以上三个问题中，第三个我们可以用 备份来解决。其他两个问题如何解决呢？


---



## 二、KAD算法与路由表

目前能解决上述问题的路由表算法有：KAD,DHT,CHORD等算法。这里由于以太坊采用了主流KAD算法，故主要总结KAD算法的原理。

__*<span data-type="color" style="color:#8C8C8C">这里YY下，为啥KAD会成为主流，据资料显示有三方面原因：</span>*__
> 1、比DHT简单;
> 2、K桶的实现比CHORD更灵活，直接调整K值大小来适应不同场景；
> 3、天生支持并发


### 距离：

KAD中引入了节点距离的概念。当我想将资源加入到集群中时，根据资源的key 寻找 相同hash值的目标节点 并未找到时，这是我们就去寻找与目标节点最近的一个节点，并将资源存在其上。为了防止该节点down机，一般寻找最近的N个节点存储数据。

这里的距离时XOR 逻辑距离，而非真实的物理距离。这就是比DHT简单的地方。
<span data-type="color" style="color:#52C41A">举例：节点A 的HASH值：     1111 0000</span>
<span data-type="color" style="color:#52C41A">    节点B 的HASH值：     1111 0011</span>
<span data-type="color" style="color:#52C41A">    距离XOR = 11110000 ^ 11110011 = 3</span>

### K桶(K-Buckets):

这为了好理解，我们假设P2P集群节点HASH值为3位长度。然后根据其二进制构建二叉树，左0右1。
如下图(为了更好理解层的概念，我将二叉树做了旋转)：

<img class="image" src="{{ site.url }}/assets/images/cipher/kad1.png" alt="">
<figcaption class="caption">augustus</figcaption>

---

如上图，假如节点的hash值有n位，那么我们的K桶就有N层。并且有N个子树。所以理论上，只要知道子树上的一个节点，就可以遍历该子树上的所有节点。

<span data-type="color" style="color:#1D39C4">假如以目标节点A为例，从倒数第一位开始不同计算：</span>
<span data-type="color" style="color:#1D39C4">第一层：倒数第一位不同： 节点包括： 110</span>
<span data-type="color" style="color:#1D39C4">第二层：倒时第二位不同： 节点包括：  100 ，  101</span>
<span data-type="color" style="color:#1D39C4">第三层：倒时第三位不同： 节点包括：  000 ，  010，   001， 011</span>
<span data-type="color" style="color:#1D39C4">第N层： 倒数第N 位不同： 节点个数有 2^n个。</span>

<span data-type="color" style="color:#1D39C4">还是以目标节点A为例，每一层到节点A的逻辑距离范围如下：</span>
<span data-type="color" style="color:#1D39C4">第一层：倒数第一位不同  XOR = [2^0, 2^1) = [1,2)</span>
<span data-type="color" style="color:#1D39C4">第二层：倒数第二位不同  XOR = [2^1,2^2) = [2,4)</span>
<span data-type="color" style="color:#1D39C4">第三层：倒数第三位不同  XOR = [2^2,2^3) = [4,8)</span>
<span data-type="color" style="color:#1D39C4">第n层： 倒数第n位不同    XOR = [2^(n-1),2^n)</span>


按照这种方式，P2P中每个节点按照K-Bucket的分层方式将其他节点保存在本地。当写入数据是，按照数据资源的KEY的HASH与目标节点的HASH值做距离计算，首先就能定位到该资源应该保存在哪一层的节点上，然后在当前层中找到与目标节点距离最近的节点，将数据保存其中。

当集群P2P的网络庞大是，保存所有的节点信息是不现实的。理论上我们只要知道每一层中的一个节点，就可以将该层中所有的节点遍历而出。故KAD中K系数就是表示 每层最大存储的节点个数，以太坊，电驴中设置为8。

加入现在我动态添加了一个节点X，理论上数据Y应该存储在X上，但是当Y存储时还没有X，这是X肯定就存在了里Y最近的几台节点上。所以此时查找时，当Y上没有找到该数据时，就去Y附近的节点依次查找下去，这样就避免了遍历所有节点查找资源，相对快速的锁定了资源。

---

所以查询方法应该包括但不限于以下(以太坊均已实现)：
FIND\_NODE:  查询某个节点邻居节点
FIND\_VALUE: 根据资源的hash值，查找可能存在于哪些邻居节点上

```
/* 查询目标节点的n个邻居节点 */
case findnodePacket:
        target := crypto.Keccak256Hash(pkt.data.(*findnode).Target[:])

        //找到离节点target最近的16个节点
        results := net.tab.closest(target, bucketSize).entries
        net.conn.sendNeighbours(n, results)
        return n.state, nil
/* 这里既可表示查询节点，也可表示查询数据，查询某资源可能存在的邻居节点 */
case findnodeHashPacket:
        results := net.tab.closest(pkt.data.(*findnodeHash).Target, bucketSize).entries
        net.conn.sendNeighbours(n, results)
        return n.state, nil
```


