---
title: Netty Asynchronous Task
tags:
  - Netty
category:
  - Netty
author: bsyonline
lede: 没有摘要
date: 2020-01-04 19:24:53
thumbnail:
---

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