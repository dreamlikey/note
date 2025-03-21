### **`ConcurrentHashMap` 如何保证线程安全？**

`ConcurrentHashMap` 通过 **CAS + 分段锁 + `volatile` 变量** 共同保证 **线程安全**，并且 **在高并发下提供更优的性能**，避免了 `synchronized` 造成的全局锁竞争问题。它的线程安全主要体现在 **数据结构设计** 和 **并发控制机制** 上。

------

## **1. `ConcurrentHashMap` 的数据结构**

### **JDK 1.7 版本**

在 **JDK 1.7** 中，`ConcurrentHashMap` 采用 **"分段锁（Segment）+ 数组 + 链表"** 的结构：

```java
Segment<K, V>[] segments; // 分段锁数组，每个 Segment 保护一部分数据
```

- **`Segment` 继承 `ReentrantLock`**，相当于小型的 `HashMap`，每个 `Segment` 内部维护一个 `HashEntry<K, V>[]` 数组。
- 访问数据时：
  - **不同的 `Segment` 可并发访问**（提高并发性能）
  - **同一 `Segment` 内部需要加锁**（保证线程安全）

### **JDK 1.8 版本**

JDK 1.8 后，`ConcurrentHashMap` 采用 **"数组 + 链表 + 红黑树 + CAS"** 的结构：

```java
Node<K, V>[] table; // 主数组
```

- **去掉了 `Segment`，改用 `synchronized` 和 CAS（无锁优化）**
- **链表长度 > 8 时转换为红黑树，提高查询性能**
- **CAS + `synchronized` 代替 `ReentrantLock`，减少锁的粒度，提高并发性能**

------

## **2. `ConcurrentHashMap` 的线程安全实现**

### **2.1 `put()` 方法的线程安全**

```java
map.put("key1", "value1");
```

#### **(1) 计算索引**

`ConcurrentHashMap` 计算 `key` 的 **哈希值**，然后找到 `table` 数组中的索引：

```java
int hash = spread(key.hashCode()); // 扰动哈希算法
int index = (table.length - 1) & hash;
```

#### **(2) 插入数据（核心并发控制）**

- 如果该位置是空的

  （即 

  ```
  table[index] == null
  ```

  ）：

  - 通过 

    CAS（无锁）插入新节点

    ：

    ```java
    if (casTabAt(table, index, null, new Node<K, V>(hash, key, value, null)))
        return;
    ```

  - CAS（Compare-And-Swap）保证多个线程同时插入时，只有一个能成功。

- 如果该位置不为空

  ：

  - 使用 `synchronized` 锁定 `Node`

    ，避免并发修改：

    ```java
    synchronized (node) {
        // 插入新节点或更新值
    }
    ```

  - **链表转换红黑树**：当链表长度超过 8 时，自动转换为 **红黑树**，提高查询性能。

------

### **2.2 `get()` 方法的线程安全**

```java
String value = map.get("key1");
```

- 读取数据时，只需 

  ```
  volatile
  ```

   保证 

  可见性

  ，不需要加锁：

  ```java
  Node<K, V> e = tabAt(table, index);
  if (e != null) {
      if (e.hash == hash && (e.key == key || key.equals(e.key)))
          return e.value;
  }
  ```

- **查询操作无锁（无竞争）**，因此是高效的。

------

### **2.3 `size()` 方法的线程安全**

`size()` 统计多个 `Node<K, V>` 的数量，需要**遍历整个数组**，并发环境下容易不准确。

**JDK 1.8 的优化**：

- `ConcurrentHashMap` 使用 **基于 `LongAdder` 的计数器** 来 **减少全局锁**，提高 `size()` 的准确性。
- 但仍然是 **估算值**，如果要求精确，需要 `sumCount()` 进行二次统计。

------

## **3. `ConcurrentHashMap` 线程安全核心机制**

| **机制**                      | **作用**                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| **CAS（Compare-And-Swap）**   | 在插入新节点时，使用 **CAS 进行无锁更新**，避免全局锁，提高并发性能 |
| **`synchronized`（JDK 1.8）** | **链表或红黑树操作时，使用 `synchronized` 保证安全**，但锁仅作用于单个桶，减少锁竞争 |
| **`volatile`**                | **`table` 数组和 `Node` 头节点使用 `volatile` 关键字**，保证可见性，避免数据不一致 |
| **红黑树优化**                | **链表长度 > 8 时转换为红黑树**，避免哈希冲突导致的查询性能下降 |
| **分段锁（JDK 1.7）**         | **JDK 1.7 版本使用 `Segment` 作为分段锁**，但 JDK 1.8 后优化为 CAS + `synchronized` |

------

## **4. `ConcurrentHashMap` vs. `HashMap` vs. `Hashtable`**

| 特性         | `ConcurrentHashMap`           | `HashMap`                | `Hashtable`                 |
| ------------ | ----------------------------- | ------------------------ | --------------------------- |
| **线程安全** | ✅ 是（CAS + `synchronized`）  | ❌ 否                     | ✅ 是（全局 `synchronized`） |
| **并发性能** | 🚀 **高（分段锁 + 无锁优化）** | 🐢 **低（需要手动加锁）** | 🐌 **极低（全局锁）**        |
| **数据结构** | **数组 + 链表 + 红黑树**      | **数组 + 链表 + 红黑树** | **数组 + 链表**             |
| **适用于**   | **高并发环境**                | **单线程或低并发**       | **已淘汰（性能差）**        |

------

## **5. 结论**

🔹 `ConcurrentHashMap` 主要通过 **CAS + `synchronized` + `volatile`** 结合，实现线程安全。
 🔹 **JDK 1.8 采用"数组 + 链表 + 红黑树"结构，优化查询效率**，避免了 `HashMap` 的哈希冲突问题。
 🔹 **读取操作无锁，写入操作仅锁定桶内链表或红黑树，避免全局锁竞争**，相比 `Hashtable` 提高并发性能。