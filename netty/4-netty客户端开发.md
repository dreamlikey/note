#### 示例代码

```java
public class TimeClient {
    public void connect(int port, String host) throws Exception {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            //客户端的启动辅助类
            Bootstrap bs = new Bootstrap();
            bs.group(group)
                .channel(NioSocketChannel.class)
                //绑定网络IO事件
                .handler(new TimeClientChannelHandler())
                //TCP参数
                .option(ChannelOption.TCP_NODELAY, true);
            ChannelFuture cf =  bs.connect(host,port).sync();
            cf.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception {
        int port = 2020;
        new TimeClient().connect(port, "127.0.0.1");
    }

 }

 class TimeClientChannelHandler extends ChannelInitializer<SocketChannel> {

     @Override
     protected void initChannel(SocketChannel socketChannel) throws Exception {
         socketChannel.pipeline().addLast(new TimeClientHandler());
     }
 }

 class TimeClientHandler extends ChannelHandlerAdapter {

    @Override
     public void channelActive(ChannelHandlerContext ctx) throws Exception {
         byte[] req = "time client message".getBytes();
         ByteBuf message = Unpooled.buffer(req.length);
         message.writeBytes(req);
         ctx.writeAndFlush(message);
     }

     @Override
     public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
         ByteBuf buf = (ByteBuf) msg;
         byte[] rep = new byte[buf.readableBytes()];
         buf.readBytes(rep);
         String body = new String(rep, "UTF-8");
         System.out.println("Time client receive body : " + body);
     }

     @Override
     public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
         ctx.close();
     }
 }
```

