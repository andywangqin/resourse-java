###1、基础概念
RPC（Remote Procedure Call Protocol），在各大互联网公司中被广泛使用，如阿里巴巴的hsf、dubbo（开源）、Facebook的thrift（开源）、Google grpc（开源）、Twitter的finagle（开源）等。
RPC 可基于 HTTP 或 TCP 协议，Web Service 就是基于 HTTP 协议的 RPC，它具有良好的跨平台性，但其性能却不如基于 TCP 协议的 RPC。会两方面会直接影响 RPC 的性能，一是传输方式，二是序列化。
众所周知，TCP 是传输层协议，HTTP 是应用层协议，而传输层较应用层更加底层，在数据传输方面，越底层越快，因此，在一般情况下，TCP 一定比 HTTP 快。就序列化而言，Java 提供了默认的序列化方式，但在高并发的情况下，这种方式将会带来一些性能上的瓶颈，于是市面上出现了一系列优秀的序列化框架，比如：Protobuf、Kryo、Hessian、Jackson 等，它们可以取代 Java 默认的序列化，从而提供更高效的性能。
---
####1.1、RPC调用的流程

![](image/640.png)

1. 服务消费方（client）调用以本地调用方式调用服务；
2. client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；
3. client stub找到服务地址，并将消息发送到服务端；
4. server stub收到消息后进行解码；
5. server stub根据解码结果调用本地的服务；
6. 本地服务执行并将结果返回给server stub；
7. server stub将返回结果打包成消息并发送至消费方；
8. client stub接收到消息，并进行解码；
9. 服务消费方得到最终结果。

RPC的目标就是要2~8这些步骤都封装起来，让用户对这些细节透明。

###2、怎么做到透明化远程服务调用？
怎么封装通信细节才能让用户像以本地调用方式调用远程服务呢？对java来说就是使用代理！java代理有两种方式：

- jdk 动态代理
- 字节码生成

尽管字节码生成方式实现的代理更为强大和高效，但代码维护不易，大部分公司实现RPC框架时还是选择动态代理方式。

下面简单介绍下动态代理怎么实现我们的需求。我们需要实现RPCProxyClient代理类，代理类的invoke方法中封装了与远端服务通信的细节，消费方首先从RPCProxyClient获得服务提供方的接口，当执行helloWorldService.sayHello(“test”)方法时就会调用invoke方法。
```
```

###3、怎么对消息进行编码和解码？
####3.1、确定消息数据结构

####3.2、序列化
目前互联网公司广泛使用Protobuf、Thrift、Avro等成熟的序列化解决方案来搭建RPC框架，这些都是久经考验的解决方案。

####3.3、通信
前有两种常用IO通信模型：1）BIO；2）NIO。一般RPC框架需要支持这两种IO模型。

使用java nio方式自研，这种方式较为复杂，而且很有可能出现隐藏bug，但也见过一些互联网公司使用这种方式；

基于mina，mina在早几年比较火热，不过这些年版本更新缓慢；

基于netty，现在很多RPC框架都直接基于netty这一IO通信框架，省力又省心，比如阿里巴巴的HSF、dubbo，Twitter的finagle等。

####3.4、消息里为什么要有requestID？
requestID（requestID必需保证在一个Socket连接里面是唯一的），一般常常使用AtomicLong从0开始累计数字生成唯一ID；

###4、如何发布自己的服务？


###9、案例
Spring：它是最强大的依赖注入框架，也是业界的权威标准。
Netty：它使 NIO 编程更加容易，屏蔽了 Java 底层的 NIO 细节。
Protostuff：它基于 Protobuf 序列化框架，面向 POJO，无需编写 .proto 文件。
ZooKeeper：提供服务注册与发现功能，开发分布式系统的必备选择，同时它也具备天生的集群能力

### 参考资料：

- [RPC原理及RPC实例分析](http://mp.weixin.qq.com/s?__biz=MjM5NzMyMjAwMA==&mid=2651477714&idx=1&sn=484810a4c8b7fe8ce066d822a2c8d238&chksm=bd253aad8a52b3bb106fed92491973367e83c7b7c9e43bda5c0da4a1e43abb7fc401f0a1a8a9&scene=0#wechat_redirect)