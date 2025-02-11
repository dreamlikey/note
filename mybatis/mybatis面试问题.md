### 什么是mybatis

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录

### JDBC缺点

1、硬编码

注册驱动、获取连接

sql语句要手写



2、操作繁琐

手动设置参数

手动封装结果集



### 谈谈你对mybatis的理解



### 介绍下mybastis工作原理



### 介绍下mybatis的缓存设计



### 谈谈mybatis的分页原理

1.分页的原理

2.mybatis中分页的实现

mybatis中分页两种实现：

1.逻辑分页 rowbounds

2.物理分页 拦截器实现



### 谈谈sqlsession的安全问题

sqlsessionfactory是线程安全

DeaultSqlSession不是线程安全的【因为类中存在多个方法进行修改的成员变量，这些方法也没有加锁】，



### Spring中是如何解决DeaultSqlSession的数据安全问题的





sqlsessionTemplate管理session的生命周期，基于spring事务对session进行关闭、回滚、提交操作，sqlsessionTemplate实现了sqlsession接口，在spring中使用mapper接口操作selectOne等方法时，实际上调用的是SqlSessionTemplate提供的代理方法，

```java
@Override
public <T> T selectOne(String statement, Object parameter) {
  return this.sqlSessionProxy.<T> selectOne(statement, parameter);
}
```

代理方法中每次调用会创建一个新的sqlsession，如果该线程有事务从当前事务ThreadLocall中拿到sqlsessioon。

