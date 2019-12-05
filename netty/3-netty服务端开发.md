#### 示例代码

```java
public class TimeServer {

    /**
     * Bind.
     * @param port the port
     * @throws Exception the exception
     */
    public void bind(int port) throws Exception{
        //NIO线程组，接收客户端连接
        EventLoopGroup bossGroup   = new NioEventLoopGroup();
        //NIO线程组，进行SocketChannel网络读写
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            //启动服务端的辅助启动类
            ServerBootstrap b = new ServerBootstrap();
            b.group(bossGroup, workerGroup)
                //声明channel为NioServerSocketChannel
                .channel(NioServerSocketChannel.class)
                //设置TCP参数
                .option(ChannelOption.SO_KEEPALIVE, true)
                //绑定网络I/O事件
                .childHandler(new ChildChannelHandler());
            //绑定监听端口，sync同步阻塞等待绑定完成
            ChannelFuture cf = b.bind(port).sync();
            //阻塞等待服务端链路关闭
            cf.channel().closeFuture().sync();
        } finally {
            //优雅退出，释放线程池资源（服务器链路关闭之后执行）
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) {
        int port = 2020;
        try {
            new TimeServer().bind(port);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

/**
 * The type Child channel handler.
 */
class ChildChannelHandler extends ChannelInitializer<SocketChannel> {

    @Override
    protected void initChannel(SocketChannel socketChannel) {
        socketChannel.pipeline().addLast(new TimeServerHandler());
    }

}

/**
 * The type Time server handler.
 */
class TimeServerHandler extends ChannelHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        super.channelActive(ctx);
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //读取数据
        ByteBuf byteBuf = (ByteBuf) msg;
        byte[] req = new byte[byteBuf.readableBytes()];
        byteBuf.readBytes(req);
        String body = new String(req, "UTF-8");
        System.out.println("Time server receive order : "+ body);

        //发送数据
        byte[] resp = "time server message".getBytes();
        ByteBuf message = Unpooled.buffer(req.length);
        message.writeBytes(resp);
        ctx.writeAndFlush(message);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //异常释放ChannelHandlerContext资源
        ctx.close();
    }
}
```



##### NioEventLoopGroup

NIO线程组，代码中声明了两个线程组，**boosGroup接收客户端连接，workerGroup处理SocketChannel网络读写**

##### ServerBootstrap

启动服务端的辅助类（allows easy bootstrap of {@link ServerChannel}），简化服务端的初始化和启动

##### Option

TCP参数

##### ChannelHandlerAdapter

网络IO事件，事件中进行数据处理、异步调用业务处理、日志记录等



##### ByteBuf