### 对象组成

openjdk hotspot 官方文档http://openjdk.java.net/groups/hotspot/

java对象保存在内存中，由三部分组成

- 对象头
- 实例数据
- 对象填充字节



#### 代码

```java
public class Test {
    byte i = 0;
    short s = 0;
    long j = 0;
    int k = 1;
    public static void main(String[] args) {
        ClassLayout layout = ClassLayout.parseInstance(new Test());
        System.out.println(layout.toPrintable());
    }
}
```



对象信息

12byte的对象头信息，

实例变量所分配的内存空间大小，

最后JVM补齐的5个字节，

为了确保对象分配的堆内存空间为8的整数倍，27byteJVM补齐5byte实际占用32byte

```java
Test object internals:
 OFFSET  SIZE    TYPE DESCRIPTION                               VALUE
      0     4         (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4         (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4         (object header)                           54 c7 00 20 (01010100 11000111 00000000 00100000) (536921940)
     12     4     int Test.k                                    1
     16     8    long Test.j                                    0
     24     2   short Test.s                                    0
     26     1    byte Test.i                                    0
     27     5         (loss due to the next object alignment)
Instance size: 32 bytes
Space losses: 0 bytes internal + 5 bytes external = 5 bytes total
```



object header 对象头

​	4     int Test.k  				实例数据

loss due to the next object alignmen 对象填充数据



### 对象头

#### 定义

openjdk对jvm规范对象头定义如下：



每个gc管理的堆对象开头的公共结构。(每个oop都指向一个对象标头。)包括堆对象的布局、类型、GC状态、同步状态和标识哈希码的基本信息。由两个词组成（mark word,klass pointer）。在数组中，它后面紧跟着一个长度字段。注意，Java对象和vm内部对象都有一个通用的对象头格式。



#### 两个词

**mark word**

The first word of every object header. Usually a set of bitfields including synchronization state and identity hash code. May also be a pointer (with characteristic low bit encoding) to synchronization related information. During GC, may contain GC state bits.

**klass pointer**

每个对象标头的第二个单词。指向描述原始对象的布局和行为的另一个对象(元对象)。对于Java对象，“klass”包含一个c++风格的虚拟表（“vtable”）。