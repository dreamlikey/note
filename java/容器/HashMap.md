## Hash算法

高16位与低16位异或运算

```java
 static final int hash(Object key) {
        int h;
     	//高16位与低16位异或运算
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```



## Hash冲突

链地址法

链表转红黑树

## 扩容方案





## 线程安全

### 线程不安全

### 解决方案

HashTable

SynchronizeMap

ConcurrentHashMap



### fast-fail

