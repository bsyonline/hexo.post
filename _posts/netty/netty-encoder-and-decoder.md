---
title: Netty Encoder and Decoder
tags:
  - Netty
category:
  - Netty
author: bsyonline
lede: 没有摘要
date: 2020-01-04 19:45:38
thumbnail:
---


我们在网络编程时，数据在网络中通常都是以字节码的形式传输的，而到了应用程序，需要将字节码转化成业务能够理解的形式，这个过程就是编解码。Netty 作为一个通用的网络编程框架，也提供了一些通用的抽象来帮助我们实现编解码工作。
编解码是编码和解码的统称，我们在发送数据之前，需要将数据进行编码，在收到数据后，在按照对应的方式解码。我们可以通过一个例子来了解 Netty 中的编解码器。
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