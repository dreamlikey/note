

#### 概念

**compare and swap比较并交换，它是一条CPU并发原语**

**它的功能是比较某个内存空间的值是否是预期值，如果是则修改为新的值，不是返回失败，这个过程是原子性的**

比较并交换：类似于乐观锁的版本号

java通过unsafe中的native方法获取操作系统资源直接访问并操作所在内存地的内存中的数据



几个疑问：

- 为什么CAS是原子性的
- 为什么用CAS而不是Synchrnized



#### 底层实现



##### Java CAS执行过程

**AtomicInteger原子int操作举例**



java.util.concurrent.atomic.AtomicInteger#getAndAdd

```java
private static final Unsafe unsafe = Unsafe.getUnsafe();
    //内存地址偏移量
	private static final long valueOffset;
    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
//其它线程可见
private volatile int value;

public final int getAndAdd(int delta) {
    	//this 当前对象
    	//valueOffset 内存偏移量，要修改数据的所在的内存地址
        return unsafe.getAndAddInt(this, valueOffset, delta);
    }

```

**valueOffset表示该变量值在内存中的偏移量，Unsafe通过内存地址偏移获取数据**



**sun.misc.Unsafe#getAndAddInt**

```java

public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            //读取主内存的值（主内存拷贝一个副本到当前线程的工作内存）
            //var1是当前原子操作的对象，包含当前value的值，value变量被volatile修饰对其它线程可见，如果当前线程执行交换之前被其它线程修改了，当前线程总是能看到
            var5 = this.getIntVolatile(var1, var2);
        //比较并交换，CAS原语，原子操作
        //不是期望值，轮询比较，直到交换成功
        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
        return var5;
    }

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

**执行步骤**

1. 1号CPU线程A，2号CPU线程B对同一变量value=1进行修改操作
2. 线程A、B分别从主内存空间读取变量值到各自工作内存
3. 1号CPU将线程A挂起执行其它任务，2号CPU线程B继续执行，比较主内存中变量是否是预期值（预期值就是读取到变量值），如果是将变量值修改value+1更新到主内存中，value=2
4. 1号CPU唤醒线程A，读取主内存中变量（value=2）并与期望值进行比较（value=1），与期望值不同，线程A本次修改失败，**于是重新读取重新比较**
5. 线程A重新获取value值，由于变量value被volatile修饰，其它线程对它的修改，线程A总是可见，线程继续执行compareAndSwapInt读取并比较替换，直到成功



##### Unsafe

**jdk底层提供的类，Unsafe类中的方法都是native修饰的本地方法，这些方法可以直接调用操作系统底层资源执行相关任务**

rt.jar包中

Unsafe是CAS核心类，由于Java方法无法直接访问底层操作系统，需要通过native本地方法来访问（jvm本地方法栈就是保存的navtive方法的信息），基于Usafe类可以直接操作特定的内存数据。Unsafe类存在sun.misc包中，其内部方法操作可以像C的指针一样直接操作内存，**Java中CAS操作的执行依赖于Unsafe方法**。



##### 原子性

**疑问：为什么cas为什么是原子性的**

调用Unsafe类中的CAS(compareAndSwap*)方法，JVM会编译成**CAS汇编指令**，这是一种完全依赖于硬件的功能，通过它来实现原子性操作。由于CAS是系统原语，原语是属于操作系统用语，是由若干条指令组成，用语完成某个功能的一个过程，并且【**原语的执行必须是连续的在执行过程中不允许被中断，也就是说是CAS是一条CPU原子指令，不会造成数据不一致问题**】



CAS底层的汇编指令是lock cmpxchg，lock是操作系统底层锁，保证了原子性

##### 自旋（自循环）

比较失败，再次进行变量读取并比较交换，这就叫自旋



#### 缺点

##### 长时间轮询

在CAS底层实现中，如果主内存中的值不是期望值，则轮询比较（do while），直到修改成功

高并发量的情况下，可能长时间轮询，给CPU带来比较高的CPU的开销



##### 只能保证一个共享变量的原子操作



##### ABA问题

CAS会导致“ABA问题”。
**CAS算法实现一个重要前提需要取出内存中某时刻的数据并在当下时刻比较并替换，那么在这个时间差类会导致数据的变化**
比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且线程two进行了一些操作将值变成了B,
然后线程two又将V位置的数据变成A，这时候线程one进行CAS操作发现由存中仍然是A，然后线程one操作成功。
**尽管线程one的CAS操作成功，但是不代表这个过程就是没有问题的**



###### ABA问题解决方式

原子引用+版本号AtomicStampedReference



#### 原子引用

java.util.concurrent.atomic.AtomicReference





#### 总结

cas --> unsafe --> 底层原理 --> ABA问题 --> 原子引用 -->