5中常用数据类型是**Redis对外暴露的数据类型，用于API操作**，组成他们的是其它底层基础数据结构



## API数据类型



#### 5种常用数据类型

字符串String、列表List、集合Set、有序集合Sorted Set、散列表Hash



#### 3特殊数据类型

bitmap

geospatial

Geo 类型，该功能可以推算出地理位置信息，两地之间的距离

hyperloglog 



## 底层数据结构

简单动态字符串SDS

inSet

dict

QuiklList

ZipList

SkipList

RedisObject





#### 简单动态字符串SDS

```c
struct sdshdr{
     //记录buf数组中已使用字节的数量
     //等于 SDS 保存字符串的长度
     int len;
     //记录 buf 数组中未使用字节的数量
     int free;
     //字节数组，用于保存字符串
     char buf[];
}
```

C字符串的区别，总结的说就是SDS效率更高、更安全

1、获取字符串长度，C字符串是字符数组，需要遍历整个数组才能获取长度。二SDS只需要读取len的值

2、二进制安全，C字符串除了末尾之外不能包含空字符，否则最先被读入的空字符认为是字符串结尾，这些限制使得C字符串只能保存文本数据，而不能保存图片、音频、压缩文件这样的二进制数据。

redis中不是靠空字符来判断字符串结束的，而是通过len属性，对于SDS来说即使读取空字符也是可以的。



#### 链表

List

#### 字典

#### 跳表 skip list

SkipList的特点:
1、跳跃表是一个双向链表，每个节点都包含score和ele值

2、节点按照score值排序，score值一样则按照ele字典排序

3、每个节点都可以包含多层指针，层数是1到32之间的随机数

4、不同层指针到下一个节点的跨度不同，层级越高，跨度越大

5、增删改查效率与红黑树基本一致，实现却更简单

#### 整数集合

#### 压缩列表

#### RedisObject

![image-20241101150331932](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20241101150331932.png)