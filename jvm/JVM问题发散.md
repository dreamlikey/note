GC次数频繁，你会怎么办？

首先分析GC日志，minor gc/major gc 那种gc频繁，结合工具查看(gcviewer)

1)适当增加堆内存

2）垃圾收集器是否选择合适  G1

3）如果是G1  

​	最大停顿时间是否过于严格（最大停顿时间太短，GC不充分会造成频繁GC）

​	堆内存的使用率是否过低（默认45%）





CPU持续飙升？

可能是程序存在死循环、GC太过频繁

排查：

1、top查看CPU占用最高的进程（进程id）

2、jstack/jinfo查看进程中占用资源最多的线程





## 扩展

### 内存泄漏和内存溢出的区别？

内存泄漏：内存空间没有被释放回收，导致这块内存一直被占用，可用的空间越来越少，最终可能造成内存溢出

内存溢出：

内存不足

存储内存的数据超过了制定内存空间大小，导致覆盖了其他正常数据，容易造成程序异常。

### 方法区回收的内存主要是什么内容？

没有使用的类信息、没有使用的常量

### 类信息什么时候能被回收呢？

1、堆里面不再有该对象

2、加载该类的ClassLoader已经被回收

3、java.lang.Class对象也不再有任何地方引用



### 不可达的对象一定会被回收吗？

不是

finalize() 自我拯救

### Yong Gc会stop the world吗？

会