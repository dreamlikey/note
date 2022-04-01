## 什么是ThreadLocal

ThreadLocal提供线程的局部变量，每个访问该变量的线程都有自己独立的副本，适用于线程间隔离、方法间传递的场景。



## 原理

Thread类中有个ThreadLocalMap类型的成员变量threadLocals ，意味着每个线程都有一个ThreadLocalMap，

ThreadLocalMap是一个类似于hashMap的结构ThreadLocal对象的hashcode作为key，value是ThreadLocal的Function实现。当通过ThreadLocal的get()方法使用定义好的funtion时，第一步判断当前thread的threadLocals是否为空，为空 创**建新的ThreadLocalMap并将当前ThreadLocal实例put进ThreadLocalMap中，也就是说每个线程都有自己独有的ThreadLocalMap**



### Thread

Thread类中有个ThreadLocalMap类型的成员变量threadLocals ，意味着每个线程都有一个ThreadLocalMap

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```



### ThreadLocalMap

ThreadLocalMap是一个类似于HashMap的结构，底层数据是Entry[]数组，通过开放地址发解决hash冲突，

hash散列通过斐波那契散列法来保证哈希表的离散度。key - ThreadLocal对象，value - function实现。

Entry

```java
/**
 * The entries in this hash map extend WeakReference, using
 * its main ref field as the key (which is always a
 * ThreadLocal object).  Note that null keys (i.e. entry.get()
 * == null) mean that the key is no longer referenced, so the
 * entry can be expunged from table.  Such entries are referred to
 * as "stale entries" in the code that follows.
 */
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}

/**
 * The table, resized as necessary.
 * table.length MUST always be a power of two.
 */
private Entry[] table;

// ThreadLocal为key
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }

```





### ThreadLocal

#### 初始化



#### get()

#### set()

#### remove()

#### threadLocalHashCode

ThreadLocal的散列算法

`0x61c88647` 与一个神奇的数字产生了关系，它就是 `(Math.sqrt(5) - 1)/2`。也就是传说中的黄金比例 0.618（0.618 只是一个粗略值），即`0x61c88647 = 2^32 * 黄金分割比`。同时也对应上了上文所提到的斐波那契散列法。总结下来看，`ThreadLocal` 中使用了斐波那契散列法，来保证哈希表的离散度。而它选用的乘数值即是`2^32 * 黄金分割比`

```java
/**
     * The next hash code to be given out. Updated atomically. Starts at
     * zero.
     */
    private static AtomicInteger nextHashCode =
        new AtomicInteger();

    /**
     * 2^32
     */
    private static final int HASH_INCREMENT = 0x61c88647;

    /**
     * Returns the next hash code.
     */
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    // hash
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```

### 









## 内存泄漏