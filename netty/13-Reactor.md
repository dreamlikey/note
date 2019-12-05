### Reactor模式角色

reactor有5种角色

#### Handle

句柄或描述符，他本质上是一种资源，由操作系统提供；该资源表示一个个的事件，事件可能来源于外部，也可能来源于内部。外部事件比如 客户端连接、客户端发送数据，内部事件比如操作系统的定时器事件等，Handle是事件的发源地。



#### Synchronous Event Demultiplexer

同步事件分离器，阻塞等待绑定在句柄上的事件，当可以在不阻塞的情况下操作句柄时返回；

例如I/O的多路复用select、poll、epoll就是这种实现方式，进程阻塞于select调用，等待一个或多个socket变为可读（等待方式可能是轮询、通知），状态变为可读之后对可读socket进行的数据进行处理；java nio的selector就是这个角色

#### Initiation Dispatcher

初始化事件分发器，作用是用于注册、删除和分发事件。事件同步分离器阻塞等待新事件的发生，当它检测到新事件发生后，通知事件分发器（Initiation Dispatcher ）回调特定的事件处理器（event handlers ）。



#### Event Handler 

事件处理器，对事件进行逻辑处理，NIO没有提供事件处理器，需要开发者自己实现，Netty框架对这一点进行了优化，提供了满足各种需求的事件处理器XXHandlerAdapter供使用或重写逻辑。



#### Concrete Event Handler 

具体的事件处理