---
title: Netty In Action
tags:
  - Netty
category:
  - Netty
author: bsyonline
lede: 没有摘要
date: 2099-05-05 14:00:50
thumbnail:
---


[netty 介绍](../../../../2020/01/04/netty-introduction/)
[netty IO 模型](../../../../2020/01/04/netty-io-model/)
[netty 异步任务](../../../../2020/01/04/netty-asynchronous-task/)
[netty 核心概念](../../../../2020/01/04/netty-core-concepts/)
[netty 编解码器](../../../../2020/01/04/netty-encoder-and-decoder/)
[netty handler 链调用机制](../../../../2020/01/04/netty-handler-chain/)
[netty 长连接](../../../../2020/01/04/netty-websocket-long-connection/)
[netty 粘包与拆包](../../../../2020/01/04/packet-sticking-and-unpacking/)

# Part I. 基础

[Java NIO 详解](../../../../2016/09/21/java-nio-introduction/)
[零拷贝](../../../../2020/01/04/zero-copy/)

## Part II. Netty In Action

### 1. 介绍



### 2. I/O 模型

在传统的 JavaIO 网络编程中，我们使用的是阻塞 IO 模型，即一个线程只对应一个客户端，单个线程负责客户端的连接及读写事件。

<img src="https://s2.ax1x.com/2020/02/27/3axaVA.png" alt="3axaVA.png" border="0"  style="width:700px"/>

显而易见，阻塞 IO 模型具有诸多缺点：

1. 每个客户端都需要有一个线程，并且没有 IO 操作也不能释放，导致消耗大量的服务器资源；
2. 服务器资源有限，无法承载大量的客户端请求；
3. 线程间等待、唤醒，切换上下文会严重的消耗 cpu 资源。 

所以，针对阻塞问题，Java NIO 引入了事件驱动模型。

<img src="https://s2.ax1x.com/2020/02/27/3axdUI.png" alt="3axdUI.png" border="0" style="width:750px" />

在 NIO 模型中，使用 selector 来维护客户端和服务器之间的通道 channel ，当有事件触发，selector 根据 selectKey 找到对应的通道进行处理，这样解决了线程阻塞的问题，但是这个模型相对复杂，编程难度比较高。

Reactor 模式是事件驱动的一种实现，它包含两个部分：reactor 和 handler 。reactor 使用单线程循环监听连接事件，并将请求分发给相应的 handler 进行处理，处理完成通过回调程序返回结果。下图就是**基本 reactor 模型**。

<img src="https://s2.ax1x.com/2020/02/27/3axWan.png" alt="3axWan.png" border="0" style="width:750px" />

和 NIO 模型类似，Reactor 模型让多个客户端复用一个 acceptor ，减少了阻塞造成的线程资源浪费。在没有连接事件时，线程可以进行 handler 处理。 处理完成后，线程不用销毁，可以处理新的分发任务，提高系统的处理能力。虽然减少了线程连接时的阻塞消耗，但是一个线程无法同时处理 accept 和 handler ，所以这种基本模型不适合 handler 业务比较耗时的场景。
为了能够充分的利用系统资源，又对 reactor 模型进行了改进，这就是**单 reactor 多线程模型**，如下图所示。

<img src="https://s2.ax1x.com/2020/02/27/3ax6Kg.png" alt="3ax6Kg.png" border="0" style="width:800px" />

在多线程模式中，handler 只负责响应事件，收到事件后将任务分配给线程池中的工作线程，这样任务处理是异步的，处理完成后由回调函数返回结果，不会影响线程处理新的 accept 事件，这样就进一步减少了线程阻塞。但是在单 reactor 多线程模型中，由于 reactor 是既要处理连接，又要将请求分发给 handler ，所以请求量大时，会有性能瓶颈，因此对进一步改进出现了**主从 reactor 模型**。

<img src="https://s2.ax1x.com/2020/02/27/3axD8f.png" alt="3axD8f.png" border="0" style="width:800px" />

主从 reactor 模型是在单 reactor 多线程模型基础上，对 reactor 进行了功能区分，主 reactor 只负责处理 accept 事件，接收到请求后将连接分配给子 reactor 。 子 reactor 负责将事件分发给 handler，handler 再将事件分发给 worker 线程池中的线程来处理，处理结果由回调方法返回，这样就进一步提高了服务器的处理能力。

Netty 模型是在主从 reactor 模型的基础上演化而来的，如下图所示。

<img src="https://s2.ax1x.com/2020/02/27/3axgbj.png" alt="3axgbj.png" border="0"  style="width:550px"/>

BossGroup 相当于主 reactor， WorkerGroup 相当于子 reactor 。每个 group 中都有多个不断循环处理事件的线程 NioEventLoop 。BossGroup 只处理 accept 事件，将 WorkerGroup 中的 NioEventLoop 注册到 selector 。WorkerGroup 中的 NioEventLoop 负责处理读写事件。每个 NioEventLoop 中都维护了一个 pipeline ，用来维护 handler 。 

### 3. 核心概念

Channel

对 socket 的封装，包含了一组 API ，简化了直接操作 socket 的编程复杂性。

EventLoopGroup

是一个 EventLoop 池，包含很多个 EventLoop 。Netty 为每一个 Channel 分配了一个 EventLoop ，用于处理用户连接请求，用户请求处理等事件。EventLoop 由一个线程驱动，在生命周期内只会绑定一个线程，让该线程处理这个 Channel 的所有的 I/O 事件。一个 Channel 只能绑定一个 EventLoop ，但是一个 EventLoop 可以绑定多个 Channel 。

ServerBootStrap



ChannelHandler



ChannelPipeline



ChannelFuture



### 4. 执行流程



### 5. 编解码器



### Handler 链调用机制



### 异步任务

Netty 的 handler 在处理消息时，如果业务比较耗时，响应会阻塞直到业务处理完成。在这样的场景下，通常会将业务处理提交到任务队列异步执行，而让 handler 可以快速返回。
比如可以将业务提交到 EventLoop 中的 taskQueue 。

```
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
	ByteBuf byteBuf = (ByteBuf) msg;
	System.out.println("收到客户端请求：" + byteBuf.toString(CharsetUtil.UTF_8));
	// 提交到eventLoop的tackQueue
	ctx.channel().eventLoop().execute(new Runnable() {
		@Override
		public void run() {
			myTask1();
		}
	});
}
```

还可以提交到定时任务队列中。

```
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
	ByteBuf byteBuf = (ByteBuf) msg;
	System.out.println("收到客户端请求：" + byteBuf.toString(CharsetUtil.UTF_8));
	// 提交到eventLoop的scheduledTaskQueue
	ctx.channel().eventLoop().schedule(new Runnable() {
		@Override
		public void run() {
			myTask2();
		}
	}, 0, TimeUnit.SECONDS);
}
```

任务可以提交多个，如果提交了多个任务，那么任务会在队列中排队，顺序执行。如果同时提交了 taskQueue 和 scheduledTaskQueue ，任务不会并行执行。

### 长连接



### 粘包与拆包

> TCP 协议是基于流的，数据没有边界，需要应用自己定义边界。UDP 是有数据边界的。

我们在网络编程时，数据在网络中通常都是以二进制字节码的形式传输的，在基于 TCP 网络传输过程中，通常会通过缓冲区来缓存数据，如果一次请求发送的数据量较小，则缓冲区会存放多个请求的数据，又或者如果一次发送的数据超过了缓冲区大小，则会分多次发送，TCP 协议会将用户真正要发送的数据根据当前缓存的实际情况对其进行拆分或重组，转换成网络传输的 Frame ，就产生了粘包与拆包的问题。

在 Netty 中，解决粘包与拆包的问题就是编解码的问题。在发送方将 ByteBuf 中的数据拆分或重组为 Frame ，接收方将接收到的 Frame 中的数据恢复成 BtyeBuf ，这个过程就是编解码。编解码是编码和解码的统称，在发送数据之前，需要将数据进行编码，在收到数据后，在按照对应的方式解码。Netty 作为一个通用的网络编程框架，也提供了一些通用的抽象来帮助我们实现编解码工作。我们可以通过一个例子来了解 Netty 中的编解码器。

客户端和服务器之间通过 channel 进行通信，数据发送到 channel 叫做出站，从 channel 取出数据叫做入站。因此，首先客户端发送一个 Long 类型的数据，出将 Long 类型转换成 ByteBuf ，这个过程是编码，入站从 channel 中读取 ByteBuf 转换成 Long 这个过程是解码。我们需要实现一个 Long 到 ByteBuf 的编码器和一个 ByteBuf 到 Long 的解码器。
编码器比较简单，直接使用 ByteBuf 的 writeLong 发送数据。

```
public class MyLongToByteEncoder extends MessageToByteEncoder<Long> {
    @Override
    protected void encode(ChannelHandlerContext ctx, Long msg, ByteBuf out) throws Exception {
        System.out.println("MyLongToByteEncoder的encode方法执行，将" + msg + "(Long)转成Byte");
        out.writeLong(msg);
    }
}
```

入站解码的时候需要按照 Long 类型的格式来读取数据，所以首先需要判断数据是否够 8 个字节，如果够 8 个字节，才能正确处理。

```
public class MyByteToLongDecoder extends ByteToMessageDecoder {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        System.out.println("MyByteToLongDecoder的decode执行，将Byte转成Long");
        if (in.readableBytes() >= 8) {
            out.add(in.readLong());
        }
    }
}
```

写完之后在将编码器加入到客户端的 pipeline 中，将解码器加入到服务端的 pipeline 中。
由于 Netty 提供了一些抽象类，我们只需要实现这些类来重写我们自己实现，就可以完成相应的功能，所以整个过程编码相当简单。正如我们看到的，我们自己的 Encoder 继承 MessageToByteEncoder ，这是一个 Netty 提供的抽象类。

```
public abstract class MessageToByteEncoder<I> extends ChannelOutboundHandlerAdapter {
}
```

它继承了 ChannelOutboundHandlerAdapter ，对应 bind、connect、read、write 等操作都已经实现好了，我们只需要根据自己的需要重写 encode 方法即可。根据名字我们可以知道，通过 encode 方法可以将 message 转成 byte 。encode 方法有三个参数，ChannelHandlerContext ctx 是上下文，I msg 是需要 encode 的数据， ByteBuf out 是转换之后的 ByteBuf 对象。MessageToByteEncoder 接收一个泛型用于指定将什么类型的数据转成 byte 。

同样的道理，Decoder 继承了另一个抽象类 ByteToMessageDecoder 。ByteToMessageDecoder 又继承了 ChannelInboundHandlerAdapter

```
public abstract class ByteToMessageDecoder extends ChannelInboundHandlerAdapter {
}
```

重写 decode 方法可以将 byte 转换成我们需要的格式，比如例子中的 Long 。decode 方法也有三个参数，ChannelHandlerContext ctx 是上下文， ByteBuf in 是读取的 ByteBuf 对象，List<Object> out 用来存放解码处理后的数据，比如我们发送了两个 Long 类型的消息，那么就会每次读取 8 个长度的 byte 转成 Long 然后放到集合中，最终集合中放的就是 2 个 Long 类型的数据，这些数据会被传递给 pipeline 的下一个 handler 进行处理。

上边的示例中，我们使用 ByteToMessageDecoder 的 decode 方法时需要按照数据的长度进行判断，以保证数据解析正确，这样还是有一些麻烦，Netty 还提供了另外一个类可以帮助我们简化这个操作。

```
public class MyByteToLongDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) throws Exception {
        System.out.println("MyByteToLongDecoder的decode执行，将Byte转成Long");
        out.add(in.readLong());
    }
}
```

ReplayingDecoder 继承自 ByteToMessageDecoder，可以帮我们在读取缓冲区的数据之前需要检查缓冲区是否有足够的字节，而不用我们自己检查。

```
public abstract class ReplayingDecoder<S> extends ByteToMessageDecoder {
```

简单来看，继承 ByteToMessageDecoder 和继承 ReplayingDecoder 除了用户自己判断数据长度之外，其他操作都是一样的。但是使用 ReplayingDecoder 还是要注意，比如下面这个例子。

```
public class MyDecoder extends ReplayingDecoder<Void> {

    private final Queue<Integer> values = new LinkedList<Integer>();

    @Override
    public void decode(ChannelHandlerContext ctx, ByteBuf buf, List<Object> out) throws Exception {

        // A message contains 2 integers.
        values.offer(buf.readInt());
        values.offer(buf.readInt());

        // This assertion will fail intermittently since values.offer()
        // can be called more than two times!
        assert values.size() == 2;
        out.add(values.poll() + values.poll());
    }

}
```

这个 Decoder 的作用是先读取两个 int 放到队列中，如果读到 2 个 int 再将 2 个 int 都取出来相加，将结果交给下一个 handler 处理。看上去很简单，但是实际上是无法正确执行的。这和 ReplayingDecoder 的处理机制有关。简单来说 ReplayingDecoder 就是不断的读取 ByteBuf ，如果没有读到想要的数据，就抛出异常，然后捕获异常，重置位置后重新再次读取。所以上边的示例中，当第二次读取不够 int 长度时，会再次调用 decode 重新读取，直到读取了两个 int ，但是由于之前第一个 int 已经读到并加入到 Queue<Integer> 中了，当第二次读取到 2 个 int 时，Queue<Integer> 中已经有 3 个 int 了，assert values.size() == 2; 的断言就永远无法成功，所以需要在第一次读取之前先清空之前的数据才能保证程序正确执行。

除了上边的几种编解码器之外，Netty 还提供了一组编解码器 MessageToMessageEncoder 和 MessageToMessageDecoder 用于格式之间转换，使用方法和其他编解码器类似。