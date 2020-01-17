



![可重入锁-流程](image\可重入锁-流程.jpg)

### 加锁

#### aquire

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

#### park



### 释放锁



#### unpark