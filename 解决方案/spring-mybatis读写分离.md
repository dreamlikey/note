```
@ConfigurationProperties
@ConditionalOnClass
@EnableConfigurationProperties
```





https://www.jianshu.com/p/2222257f96d3

#### 读写分离

#### 声明DataSource

##### 	多数据源

##### 	兼容单数据源

#### 声明Mybatis

#### 动态切换数据源

##### 多读

##### 负载均衡

#### 问题

##### 数据一致性

同一个用户 执行 **增删改** 后 **下一秒内读**请求，读取写库

1、增删改  【方法全名 +  方法的参数 hashcode】为key 做redis缓存

2、透传userId + 方法参数 hashcode 做reids缓存

​	需要依赖透传组件



##### 事务

@Transactional   修饰的service，强制写库



没有声明事务，但是有DDL操作，DDL之后的操作强制 写库



##### 数据源异常处理





##### 复杂点

主从复制延迟

1、写操作后的读操作指定发给主库



2、二次读取

读取从机失败后再读一次从机

如果有很多二次读取，将大大增加主机读操作压力。例如，黑客暴力破解账号，会导致大量的二次读取操作，主机可能顶不住读操作的压力从而崩溃。



3、关键业务读写操作全部指向主机，非关键业务采用读写分离

分配机制