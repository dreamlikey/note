### netty socket数据传输

##### 断线重连

###### 启动重连

###### 运行中重连

##### 心跳检测

网络抖动、网络中断、操作系统中断连接，netty没有主动感知到连接断开，不能及时重连

定时向服务端发送心跳数据，多个心跳之后客户端还没有接收到服务端的响应或者长时间未向服务端发送心跳也没有接收到服务端响应，则判断连接出现问题，启动重连。

###### 心跳事件

IdleStateHandler空闲状态处理器，定时触发心跳事件

```java
//开启心跳
ch.pipeline().addLast("ping", new IdleStateHandler(60, 20, 60 * 10, TimeUnit.SECONDS));

/**
     * @see #IdleStateHandler(boolean, long, long, long, TimeUnit)
     */
    public IdleStateHandler(
        	//读事件间隔时间        写事件空闲时间         
            long readerIdleTime, long writerIdleTime, long allIdleTime,
            TimeUnit unit) {
        this(false, readerIdleTime, writerIdleTime, allIdleTime, unit);
    }
```



###### 事件触发器



申明心跳事件后，需要申明事件的触发器，用来对事件触发后进行相应的处理

```java
@Override
public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
        super.userEventTriggered(ctx, evt);
    	//如果是心跳事件
        if (evt instanceof IdleStateEvent) {
            IdleStateEvent event = (IdleStateEvent) evt;
            if (event.state().equals(IdleState.READER_IDLE)) {
                Constant.LUBAN_SOCKET_SEND_FLAG = Boolean.FALSE;
                log.error("长期没收到服务器推送数据，开始重连。。。");
                client.startClient();
            } else if (event.state().equals(IdleState.WRITER_IDLE)) {
//                log.error("长期未向服务器发送数据");
                //发送心跳包
                ctx.writeAndFlush(GPSInfoModel.GPSInfoProto.newBuilder().setType(1)
                    .setCode(Constant.LUBAN_SOCKET_SEND_AUTH_CODE).build());
            } else if (event.state().equals(IdleState.ALL_IDLE)) {
                Constant.LUBAN_SOCKET_SEND_FLAG = Boolean.FALSE;
                log.error("长期长期未向服务器发送、没收到服务器推送数据，开始重连。。。");
                client.startClient();
            }
        }
    }
```



##### 粘包问题

###### 粘包概念

多个数据包连续存储于缓存中，在对数据包进行读取时由于无法确定发送方的发送边界，而采用估测值大小进行数据读取，若双方的size不一致时就会让多个数据包粘成一个包，后一包数据的头紧接着前一包数据的尾。

###### 粘包的原因

出现粘包原因是多方面的，它既可能是发送方造成、也可能是接收方造成

发送方：

​	发送方引起的粘包是由TCP协议本身造成的，TCP为提高传输效率，收集到足够多的数据才发送一包数据。

接收方：

​	接收方引起粘包是由于处理数据不及时，接收方将数据放在接收缓冲区，如果处理数据不及时，前一个包就与后面的包粘在了一起



###### 处理方式

netty自带解码器LengthFieldBasedFrameDecoder

maxFrameLength：单包最大长度

lengthFieldOffset：数据长度偏移量

lengthFieldLength：数据长度所占字节数

lengthAdjustment：

initialBytesToStrip：取消息头数据的开始字节数



```java
 /**
     * Creates a new instance.
     *
     * @param maxFrameLength
     *        the maximum length of the frame.  If the length of the frame is
     *        greater than this value, {@link TooLongFrameException} will be
     *        thrown.
     * @param lengthFieldOffset
     *        the offset of the length field
     * @param lengthFieldLength
     *        the length of the length field
     * @param lengthAdjustment
     *        the compensation value to add to the value of the length field
     * @param initialBytesToStrip
     *        the number of first bytes to strip out from the decoded frame
     */
    public LengthFieldBasedFrameDecoder(
            int maxFrameLength,
            int lengthFieldOffset, int lengthFieldLength,
            int lengthAdjustment, int initialBytesToStrip) {
        this(
                maxFrameLength,
                lengthFieldOffset, lengthFieldLength, lengthAdjustment,
                initialBytesToStrip, true);
    }


ch.pipeline().addLast(new LengthFieldBasedFrameDecoder(100, 0, 4, 0, 4));
```

1、消息定长，报文消息固定100个字节，不够用空格补位

2、将消息分为消息头和消息体，消息头中包含消息头长度字段

3、消息分割，包尾增加回车换行符进行分割



##### 序列化

###### protobuf

Protocol buffers是一个灵活的、高效的、自动化的用于对结构化数据进行序列化的协议，优点快、码流小、操作简单、跨语言。

.proto文件

```java
syntax = "proto3";
option java_package = "com";
option java_outer_classname = "MessageProto";
message Message {  
  string id = 1;
  string content = 2;
  int32 type = 3;
  string code = 4;
}
```



![1558591036670](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1558591036670.png)

生成的协议使用方法

```java
MessageProto.Message.newBuilder().setType(1)
```

