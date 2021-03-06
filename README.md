# Demo file upload problem with micronaut / netty

When uploading files in excess of few MB micronaut leaks memory. It can be reproduced easily with files in excess of 100M 
but even with files larger than 10M it happens. 

Both completed and streaming uploads are affected.

## How to reproduce

Clone this repo.

```
> ./gradlew run

> Task :compileJava
Note: Creating bean classes for 1 type elements

> Task :run
08:43:22.895 [main] INFO  io.micronaut.runtime.Micronaut - Startup completed in 639ms. Server Running: http://localhost:9102
```


Now Upload file > 500MB ...

Tested with Postman, Firefox, Chrome and curl 

```
> curl -X POST http://localhost:9102/upload/streaming -F file=@large-file.mp4
```

After first upload:

```
08:43:43.543 [nioEventLoopGroup-1-5] ERROR io.netty.util.ResourceLeakDetector - LEAK: ByteBuf.release() was not called before it's garbage-collected. See http://netty.io/wiki/reference-counted-objects.html for more information.
Recent access records:
Created at:
        io.netty.buffer.PooledByteBufAllocator.newDirectBuffer(PooledByteBufAllocator.java:331)
        io.netty.buffer.AbstractByteBufAllocator.directBuffer(AbstractByteBufAllocator.java:185)
        io.netty.buffer.AbstractByteBufAllocator.directBuffer(AbstractByteBufAllocator.java:176)
        io.netty.buffer.AbstractByteBufAllocator.ioBuffer(AbstractByteBufAllocator.java:137)
        io.netty.channel.DefaultMaxMessagesRecvByteBufAllocator$MaxMessageHandle.allocate(DefaultMaxMessagesRecvByteBufAllocator.java:114)
        io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:147)
        io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:644)
        io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:579)
        io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:496)
        io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:458)
        io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:897)
        io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
        java.lang.Thread.run(Thread.java:748)
08:43:45.143 [nioEventLoopGroup-1-5] ERROR io.netty.util.ResourceLeakDetector - LEAK: ByteBuf.release() was not called before it's garbage-collected. See http://netty.io/wiki/reference-counted-objects.html for more information.
Recent access records:
Created at:
        io.netty.buffer.SimpleLeakAwareByteBuf.unwrappedDerived(SimpleLeakAwareByteBuf.java:143)
        io.netty.buffer.SimpleLeakAwareByteBuf.readRetainedSlice(SimpleLeakAwareByteBuf.java:67)
        io.netty.handler.codec.http.HttpObjectDecoder.decode(HttpObjectDecoder.java:305)
        io.netty.handler.codec.http.HttpServerCodec$HttpServerRequestDecoder.decode(HttpServerCodec.java:101)
        io.netty.handler.codec.ByteToMessageDecoder.decodeRemovalReentryProtection(ByteToMessageDecoder.java:502)
        io.netty.handler.codec.ByteToMessageDecoder.callDecode(ByteToMessageDecoder.java:441)
        io.netty.handler.codec.ByteToMessageDecoder.channelRead(ByteToMessageDecoder.java:278)
        io.netty.channel.CombinedChannelDuplexHandler.channelRead(CombinedChannelDuplexHandler.java:253)
        io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:362)
        io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:348)
        io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:340)
        io.netty.handler.timeout.IdleStateHandler.channelRead(IdleStateHandler.java:286)
        io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:362)
        io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:348)
        io.netty.channel.AbstractChannelHandlerContext.fireChannelRead(AbstractChannelHandlerContext.java:340)
        io.netty.channel.DefaultChannelPipeline$HeadContext.channelRead(DefaultChannelPipeline.java:1434)
        io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:362)
        io.netty.channel.AbstractChannelHandlerContext.invokeChannelRead(AbstractChannelHandlerContext.java:348)
        io.netty.channel.DefaultChannelPipeline.fireChannelRead(DefaultChannelPipeline.java:965)
        io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:163)
        io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:644)
        io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:579)
        io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:496)
        io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:458)
        io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:897)
        io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
        java.lang.Thread.run(Thread.java:748)
Uploaded upload-Maksimir 1080.mp4
```


After few more upload rounds, 5-6 with 600M file ...

```
08:45:03.482 [nioEventLoopGroup-1-5] ERROR i.m.h.s.netty.RoutingInBoundHandler - Unexpected error occurred: failed to allocate 16777216 byte(s) of direct memory (used: 3808428039, max: 3817865216)
io.netty.util.internal.OutOfDirectMemoryError: failed to allocate 16777216 byte(s) of direct memory (used: 3808428039, max: 3817865216)
        at io.netty.util.internal.PlatformDependent.incrementMemoryCounter(PlatformDependent.java:652)
        at io.netty.util.internal.PlatformDependent.allocateDirectNoCleaner(PlatformDependent.java:606)
        at io.netty.buffer.PoolArena$DirectArena.allocateDirect(PoolArena.java:764)
        at io.netty.buffer.PoolArena$DirectArena.newChunk(PoolArena.java:740)
        at io.netty.buffer.PoolArena.allocateNormal(PoolArena.java:244)
        at io.netty.buffer.PoolArena.allocate(PoolArena.java:226)
        at io.netty.buffer.PoolArena.allocate(PoolArena.java:146)
        at io.netty.buffer.PooledByteBufAllocator.newDirectBuffer(PooledByteBufAllocator.java:324)
        at io.netty.buffer.AbstractByteBufAllocator.directBuffer(AbstractByteBufAllocator.java:185)
        at io.netty.buffer.AbstractByteBufAllocator.directBuffer(AbstractByteBufAllocator.java:176)
        at io.netty.buffer.AbstractByteBufAllocator.ioBuffer(AbstractByteBufAllocator.java:137)
        at io.netty.channel.DefaultMaxMessagesRecvByteBufAllocator$MaxMessageHandle.allocate(DefaultMaxMessagesRecvByteBufAllocator.java:114)
        at io.netty.channel.nio.AbstractNioByteChannel$NioByteUnsafe.read(AbstractNioByteChannel.java:147)
        at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:644)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:579)
        at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:496)
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:458)
        at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:897)
        at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30)
        at java.lang.Thread.run(Thread.java:748)
```        