### 锁类型



#### 公平锁

按照请求的先后次序获取锁，后来的线程加入等待队列末尾，以后按照FIFO规则 从队列中取到线程获取锁



#### 非公平锁

线程先尝试获取锁，失败则采用公平锁的方式

缺点

有饥饿问题

java ReentrantLock而言默认非公平，非公平锁的有点在于吞吐量比公平锁大

#### 可重入锁

##### 理论

可重入锁又名递归锁，ReentrantLock/Synchronized就是典型的可重入锁，同一线程外层方法获得锁后，进入内层方法会自动获取锁，也即是说，**线程可以进入任何一个它拥有的锁所同步着的代码块（能访问这把锁的其它同步代码）**

##### 作用

可重入锁最大的作用就是防止死锁

##### demo

```java
/**
 * t1  send sms 线程t1获取锁
 * t1  send email 线程t1访问同一锁所同步的sendEmail代码
 * t2  send sms
 * t2  send email
 */
public class ReentrantLockTest {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> {
            phone.sendSms();
        },"t1").start();
        new Thread(() -> {
            phone.sendSms();
        },"t2").start();
    }
}

class Phone {
    public synchronized void sendSms() {
        System.out.println(Thread.currentThread().getName() + "  send sms");
        sendEmail();
    }

    public synchronized void sendEmail() {
        System.out.println(Thread.currentThread().getName() + "  send email");
    }
}
```



#### 自旋锁

SprinLock

##### 理论

尝试获取锁的线程不会立即阻塞，而是采用循环的方式尝试获取锁，这样的好处是 减少线程上下的切换，缺点是循环会消耗CPU性能

##### 作用

##### demo

```java
public class SpinLock {
    //原子引用
    AtomicReference<Thread> atomicReference = new AtomicReference<>();
    public boolean myLock() {
        Thread thread = Thread.currentThread();
        while (!atomicReference.compareAndSet(null, thread)) {
        }
        System.out.println(thread.getName()+"  get lock");
        return true;
    }
    public void myUnlock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread,null);
        System.out.println(thread.getName()+" unlock");
    }

    public static void main(String[] args) {
        SpinLock spinLock = new SpinLock();
        new Thread(() ->{
            spinLock.myLock();
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            spinLock.myUnlock();
        },"t1").start();

        new Thread(() ->{
            spinLock.myLock();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            spinLock.myUnlock();
        },"t2").start();
    }
}
```

打印结果

t1  get lock
t1 unlock
t2  get lock
t2 unlock

t1先获取锁，执行5秒，线程t2尝试获取失败，循环获取，t1解锁后，t2获取成功

#### 读写锁

读写分离，写操作独占锁，读操作共享锁，锁粒度更小，并发能力更强

#### 独占锁



#### 共享锁



#### 分段锁

#### 条件锁



### ReentrantLock



#### 使用示例

#### 类结构

![可重入锁类结构](image\可重入锁类结构.jpg)

#### 可重入

current == getExclusiveOwnerThread()

判断锁的持有者是不是当前线程，如果是锁状态+1并返回true

```java
 */
protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
     //判断锁的持有者是不是当前线程
     //是锁状态+1，获取锁
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

#### 公平

**申请锁逻辑**

1、如果锁没有被占有，判断等待队列是否有等待的线程，如果有加入队尾，如果没有尝试占有锁cas

2、如果锁被已被占有，判断持有锁的线程是否是当前线程，如果是使用次数+1，如果不是返回false

```java
java.util.concurrent.locks.ReentrantLock.FairSync#lock
final void lock() {
            acquire(1);
        }

public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}

protected final boolean tryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
     //判断锁的持有者是不是当前线程
     //是锁状态+1，获取锁
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}

```



#### 非公平

```java
java.util.concurrent.locks.ReentrantLock.NonfairSync#lock
final void lock() {
    //先尝试获取锁
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else //获取锁失败，公平锁方式获取锁
        acquire(1);
}
```



#### 自旋

尝试获取锁失败，将线程加入等待队列后再次申请锁

```java
acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
```

