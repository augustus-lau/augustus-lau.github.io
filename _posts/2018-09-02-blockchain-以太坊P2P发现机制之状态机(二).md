---
title: "以太坊P2P发现机制之状态机(二)"
layout: post
date: 2018-09-02 23:00
tag: 
- 区块链
- P2P
- 发现机制
image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1542525464833&di=418b6d4878a23db3a42ecc6e68718b0c&imgtype=0&src=http%3A%2F%2Fpic2.zhimg.com%2Fv2-1cb5c8c5d171a5545fe2159b2fa3b21a_1200x500.jpg
headerImage: true
hidden: false # don't count this post in blog pagination
description: "以太坊P2P发现机制之状态机"
category: blockchain
author: augustus
externalLink: false
---

## 一、以太坊P2P UDP节点间通信简介

在上一节中提到，节点进行数据传输之前，首先要进行节点的可信性校验。在以太坊中，节点的可信性校验是采用状态机模型实现的。

当server启动后，首先从db中加载种子节点，并且初始化种子节点的状态为unknown, 当种子节点加载到路由表Router中后，需要将节点的状态更新为verifyinit，表示节点状态需要进行初始化。之后Router会loop所有的种子节点，发送Ping请求与种子节点握手。当握手成功后，种子节点的状态会变为know状态，表示已经联通，握手完成。

在以太坊的P2P网络中，节点之前能够正常的进行数据传输，首先要保证节点状态为know状态。


## 二、StateMachine

状态机中最重要的两个概念：一是当前的状态；二是状态变更的事件。我们从这两个方面来了解以太坊中的状态机是如何实现的。

以太坊节点间可信性校验时，共有8种状态。从源码中可以解析出如下流转图：



<img class="image" src="{{ site.url }}/assets/images/cipher/statemachine.png" alt="">
<figcaption class="caption">augustus</figcaption>



* __unknown__ : 未知状态，也就是说对当前节点来说，远程节点不可知。
* __verifyinit__ : 待初始化状态，当种子节点从DB加载完成后，变更为该状态。也可以理解为待发送Ping数据的状态。
* __verifywait__ : 等待验证状态，该状态是指接收到Remote节点的Ping请求后，当前节点主动发送Ping对方请求后的状态。
* __remoteverifywait__: 等待验证状态，初次主动发起Ping请求后 ，节点等待Remote节点相应的状态。
* __known__: 握手完成状态，表示双方能开始正常通信的状态。
* __contested__: 竞争状态(重新验证状态)，表示当想路由表中添加一个有效节点后，需要从路由表中获取到时间最早的节点，并重新对其连接性进行验证。
* __unresponsive__：未响应状态(删除状态)，表示Remote节点已经下线或者无法连接，应该重路由表中删除。


如上，状态从unknown 如何转变为 known握手完成状态呢？ 在下一篇总结。


## 三、相关代码的实现

### 1、unknown状态的流转：

```
/* 节点初始化状态 */
    unknown = &nodeState{
        name: "unknown",

        /* 进入状态前的事件，清空该节点的缓存数据 */
        enter: func(net *Network, n *Node) {
            net.tab.delete(n)
            n.pingEcho = nil
            // Abort active .
            for _, q := range n.deferredQueries {
                q.reply <- nil
            }
            n.deferredQueries = nil
            if n.pendingNeighbours != nil {
                n.pendingNeighbours.reply <- nil
                n.pendingNeighbours = nil
            }
            n.queryTimeouts = 0
        },
        /* 当接收到 状态为unknown的节点的请求时的处理事件，当节点为unknown时，只接受ping 事件 */
        handle: func(net *Network, n *Node, ev nodeEvent, pkt *ingressPacket) (*nodeState, error) {
            switch ev {
            case pingPacket:
                net.handlePing(n, pkt)
                net.ping(n, pkt.remoteAddr)
                return verifywait, nil
            default:
                return unknown, errInvalidEvent
            }
        },
    }
```


### 2、verifyinit状态

```
/* 这个状态主要用在 network的刷新方法中，当节点从db中加载进来后，首先讲节点状态设置为 verifyinit(待验证) */
    verifyinit = &nodeState{
        name: "verifyinit",
        /* 流转到该状态时，首先要执行的操作，可以理解为后置拦截器 */
        enter: func(net *Network, n *Node) {
            net.ping(n, n.addr())
        },
        handle: func(net *Network, n *Node, ev nodeEvent, pkt *ingressPacket) (*nodeState, error) {
            switch ev {
            case pingPacket:
                net.handlePing(n, pkt)
                return verifywait, nil
            case pongPacket:
                err := net.handleKnownPong(n, pkt)
                return remoteverifywait, err
            case pongTimeout:
                return unknown, nil
            default:
                return verifyinit, errInvalidEvent
            }
        },
    }
```

### 3、verifywait状态

```
verifywait = &nodeState{
        name: "verifywait",
        handle: func(net *Network, n *Node, ev nodeEvent, pkt *ingressPacket) (*nodeState, error) {
            switch ev {
            case pingPacket:
                net.handlePing(n, pkt)
                return verifywait, nil
            case pongPacket:
                err := net.handleKnownPong(n, pkt)
                return known, err
            case pongTimeout:
                return unknown, nil
            default:
                return verifywait, errInvalidEvent
            }
        },
    }
```


<em>其他状态 略</em>

