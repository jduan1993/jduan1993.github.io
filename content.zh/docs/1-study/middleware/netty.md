---
title: Netty 详解
---

## 1. Netty 简介

Netty 是一个基于 Java 的异步事件驱动网络应用框架，简化了网络通信程序的开发。它提供高性能、高可靠性的异步网络通信能力，支持多种协议，广泛应用于分布式系统、高性能服务器、RPC 框架等场景。

---

## 2. Netty 架构与核心组件

### 2.1 线程模型

* **EventLoopGroup**：线程组，管理一组 EventLoop。
* **EventLoop**：负责处理 Channel 的所有 I/O 操作，单线程处理多个 Channel，避免多线程同步开销。

通常服务端有两组线程池：

* **BossGroup**：负责接受新连接。
* **WorkerGroup**：负责处理连接读写。

### 2.2 Channel

表示一个网络连接，封装了底层的 Socket 通道。
常用实现有：`NioSocketChannel`（基于 NIO），`EpollSocketChannel`（Linux epoll），`OioSocketChannel`（阻塞 I/O）。

### 2.3 ChannelPipeline 与 ChannelHandler

* **ChannelPipeline**：相当于责任链，保存一系列 ChannelHandler。
* **ChannelHandler**：处理 I/O 事件和数据，分为入站（Inbound）和出站（Outbound）两种。

消息流经 ChannelPipeline 依次传递给 ChannelHandler 进行处理，支持解码、编码、业务逻辑等。

### 2.4 Future 与 Promise

* **ChannelFuture**：异步操作结果的表示，所有 I/O 操作均异步返回 ChannelFuture。
* **Promise**：可以写入结果的 Future，便于异步操作结果通知。

---

## 3. Netty 工作流程

1. **启动 ServerBootstrap**，绑定端口。
2. **BossGroup 监听客户端连接**，收到连接后注册到 WorkerGroup 的 EventLoop。
3. **WorkerGroup 负责 I/O 读写事件**，触发 ChannelPipeline 中对应的 Handler。
4. **数据处理**：数据从网络读入 ByteBuf，经过解码器转为业务消息，业务 Handler 处理后编码返回 ByteBuf 写回客户端。
5. **异步操作**：所有 I/O 操作均非阻塞，调用后立即返回 ChannelFuture，完成时触发监听器。

---

## 4. 关键特性

* **异步非阻塞**：基于 Java NIO，提升高并发场景性能。
* **零拷贝**：通过 ByteBuf 实现零拷贝内存管理，减少 GC 压力。
* **高度可定制**：支持自定义协议编解码、各种传输协议。
* **事件驱动**：支持丰富事件回调，便于扩展。
* **跨平台支持**：支持 NIO、EPOLL、KQueue 等多种 I/O 模型。

---

## 5. 常用模块

* **ByteBuf**：高性能的缓冲区，替代 Java NIO ByteBuffer。
* **Codec**：编解码器，如 `StringDecoder`、`ProtobufDecoder`。
* **Handler**：业务逻辑处理。
* **Bootstrap / ServerBootstrap**：客户端和服务端启动类。

---

## 6. 使用场景举例

* 高性能聊天系统（IM）
* 分布式 RPC 框架（Dubbo 等）
* 实时游戏服务器
* 高并发 HTTP/2 和 WebSocket 服务
* 自定义协议通信服务

---

## 7. 简单示例

```java
public class EchoServer {
    public static void main(String[] args) throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(1);
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class)
             .childHandler(new ChannelInitializer<SocketChannel>() {
                 @Override
                 protected void initChannel(SocketChannel ch) {
                     ChannelPipeline pipeline = ch.pipeline();
                     pipeline.addLast(new StringDecoder());
                     pipeline.addLast(new StringEncoder());
                     pipeline.addLast(new SimpleChannelInboundHandler<String>() {
                         @Override
                         protected void channelRead0(ChannelHandlerContext ctx, String msg) {
                             System.out.println("Received: " + msg);
                             ctx.writeAndFlush("Echo: " + msg);
                         }
                     });
                 }
             });

            ChannelFuture f = b.bind(8080).sync();
            f.channel().closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```

---