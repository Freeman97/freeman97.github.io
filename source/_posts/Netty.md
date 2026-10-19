---
title: Netty
date: 2021-3-26 23:30:01
categories: 春招复习
tags:
- 2021春招
---

<!--more-->

- [] Netty线程模型
- [] Zero Copy

# 反应器模式
* 反应器Reactor: 负责查询IO事件。检测到一个IO事件时，将其发送给相应的Handler处理器去处理。这里的IO事件就是NIO中选择器监控的通道IO事件。
* Handler处理器：与IO事件绑定，负责IO事件的处理。例如，完成真正的连接建立、通道（channel）的读取、处理业务逻辑、负责将结果写出到通道等。

## 单线程的Reactor模式
* Reactor和Handler在同一线程中运行。
* 相对于传统的多线程Blocking-IO，免去了线程切换的成本
* 缺点：Reactor和Handler都在同一线程上执行。如果一个Handler被阻塞，其它所有的Handler都得不到执行。假如负责监听连接的Handler和负责数据输入输出的Handler都在同一个线程上，负责监听连接的Handler可能会被阻塞，此时整个服务都不能接受新的连接。
* 思考：为什么Redis使用了单线程的Reactor模式？
  * Redis是一个服务器，Redis的数据操作多数都在内存中，因此Handler的操作耗时应该非常少
  * 多个线程切换需要线程切换开销，如果Handler并不需要很长的阻塞时间，使用多线程模型可能弊大于利
  * Redis是一个数据库，如果使用多线程的IO模型，多个线程可能会对同一个数据进行操作，此时需要引入同步机制，而同步机制的开销是相当大的。使用单线程模型，天然保证了单个Redis的数据库操作的原子性。

## 多线程的Reactor模式
1. 升级Handler：使用线程池
2. 升级Reactor：引入多个Selector，将单个Reactor线程拆分为多个SubReactor线程，每个SubReactor线程使用一个Selector。

## VS 生产者消费者模式
反应器模式没有专门的队列去缓冲存储IO事件

## VS 观察者模式
在Reactor模式中一个事件绑定到了一个Handler上，每个IO事件被查询后Reactor会将事件分发给所绑定的Handler。而观察者模式中同一个事件（主题）可以被订阅过的多个观察者处理。

## Reactor的优缺点
### 优点：
* 响应快，不会被单个连接的同步IO阻塞整个服务
* 编程相对简单
* 可扩展：增加Reactor线程的个数

### 缺点：
* 增加一定复杂性
* 需要OS底层IO多路复用的支持

# Netty
## Netty中的Reactor
### NioEventLoop类
* 一个`NioEventLoop`拥有一个线程，负责一个Java NIO `Selector`上的IO事件轮询。一个Reactor可以注册多个netty channel。一个`EventLoop`相当于一个(Sub)Reactor。
* `EventLoopGroup`：线程组，多线程版本的反应器，其中的每一个`EventLoop`为一个SubReactor。

### Netty通道(Channel)
* Netty channel(TCP服务器里主流的就是`NioSocketChannel`)封装的就是Java NIO通道。Java NIO通道对应的就是OS底层的文件描述符(File Descriptor, fd)。
* 连接监听型socket描述符：接受客户端的socket连接。对应`ServerSocketChannel`(Java NIO)和`NioServerSocketChannel`(Netty)
* 传输数据型socket描述符：负责传输数据，在客户端和服务端都会有一个与一条TCP连接相对应的socket描述符进行数据传输。对应`SocketChannel`(Java NIO)和`NioSocketChannel`(Netty)
* 有接受关系的`NioServerSocketChannel`和`NioSocketChannel`可以被称为亲子(Parent - Child)通道

## Netty的入站处理和出站处理
* 通道中发生IO事件 -> 被EventLoop查询到 -> 分发给实现了`ChannelInboundHandler`处理器，并调用该处理器的对应处理方法(比如`channelRead`方法，这个事件在pipeline中可以通过pipeline上下文进行冒泡)，进行一系列业务处理 -> 如果事件超出管道顶端，则会被抛弃，也可以通过事件冒泡的方式传递到出站处理器，通常由出站处理器生成或者处理出站流量 -> 事件被出站处理器处理 -> 如果IO事件到达了出站管道的底部，则IO事件会交由与该channel相关联的线程进行处理，这个IO线程通常会执行实际的IO操作。

## Netty管道(pipeline)
* Handler和Reactor是多对多的关系，一个Handler可以处理多个Reactor上分发的IO事件，一个Reactor上可以使用多个Handler来处理IO事件。多个Handler可以组成一个pipeline。
* 一个Netty Channel拥有一条Pipeline。
* IO事件在Pipeline上进行双向流动，入站时按顺序由实现了`ChannelInboundHandler`的处理器处理，出站时按逆序由实现了`ChannelOutboundHandler`的处理器处理。一个Handler可以同时实现这两个接口，这样出入站都会参与处理。

## Netty脚手架(Bootstrap)