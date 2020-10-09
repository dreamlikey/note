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





