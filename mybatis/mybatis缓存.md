MyBatis 提供了 **一级缓存** 和 **二级缓存** 机制，用于提高查询性能，减少数据库访问次数。缓存的概念是 MyBatis 为了减少数据库操作的频繁访问，保留查询的结果，以便后续查询可以直接从缓存中获取，而不需要每次都查询数据库。下面我们来详细介绍这两种缓存。

### **一级缓存**

#### **概念**

一级缓存是 MyBatis 默认启用的缓存，它是基于 `SqlSession` 的缓存。每一个 `SqlSession` 实例都有自己的一级缓存，用来缓存当前 `SqlSession` 期间的查询结果。一级缓存的作用范围局限于当前 `SqlSession`，即如果同一个 `SqlSession` 中进行相同的查询，MyBatis 会从一级缓存中直接获取结果，而不是再去访问数据库。

#### **工作原理**

- **缓存范围**：一级缓存的作用范围是 `SqlSession`，即同一个 `SqlSession` 对象会缓存查询结果。
- **缓存刷新**：每当调用 `commit()` 或 `close()` 时，一级缓存会被清空。当 `SqlSession` 对象被关闭时，缓存会失效。
- **相同查询**：在同一个 `SqlSession` 中，发起相同的查询，MyBatis 会直接返回缓存中的数据，而不是再次访问数据库。
- **缓存失效**：当查询条件发生变化、`SqlSession` 被提交或关闭时，缓存会失效。

#### **一级缓存示例**

```
java复制编辑SqlSession session = sqlSessionFactory.openSession();
try {
    // 第一次查询，数据库执行查询操作
    User user1 = session.selectOne("com.example.UserMapper.selectUserById", 1);

    // 第二次查询，相同的查询条件，数据将从一级缓存中获取
    User user2 = session.selectOne("com.example.UserMapper.selectUserById", 1);

    System.out.println(user1 == user2);  // 输出 true，表示从缓存中获取
} finally {
    session.close();
}
```

在上述代码中，第二次查询 `selectUserById` 时会直接从一级缓存中获取数据，而不是再次访问数据库。

#### **缓存失效情况**

- **提交事务**：如果在当前 `SqlSession` 中执行了提交操作（`session.commit()`），一级缓存中的数据会失效。
- **关闭 `SqlSession`**：当 `SqlSession` 被关闭时，一级缓存中的数据会被清空。
- **手动清除缓存**：可以通过 `session.clearCache()` 方法来清空一级缓存。

### **二级缓存**

#### **概念**

二级缓存是跨 `SqlSession` 的缓存，它的作用范围是整个 `SqlSessionFactory`，即一个 `SqlSessionFactory` 中的多个 `SqlSession` 可以共享二级缓存。二级缓存可以缓存 SQL 查询的结果，减少数据库查询的次数。与一级缓存不同，二级缓存的生命周期较长，通常在 MyBatis 配置文件中启用。

#### **工作原理**

- **缓存范围**：二级缓存的作用范围是 `SqlSessionFactory`，即所有从同一个 `SqlSessionFactory` 创建的 `SqlSession` 对象共享同一个二级缓存。
- **配置启用**：需要在 MyBatis 配置文件中启用二级缓存。
- **缓存存储**：二级缓存的存储方式可以选择使用内存缓存、文件系统、数据库等。
- **缓存失效**：二级缓存的失效机制取决于缓存的策略，通常可以通过配置过期时间或手动清除缓存。

#### **二级缓存启用**

1. **在 MyBatis 配置文件中启用二级缓存**：

   ```
   xml复制编辑<configuration>
       <settings>
           <setting name="cacheEnabled" value="true"/>
       </settings>
       <mappers>
           <mapper resource="com/example/UserMapper.xml"/>
       </mappers>
   </configuration>
   ```

2. **在映射文件（Mapper XML）中配置二级缓存**：

   ```
   xml复制编辑<mapper namespace="com.example.UserMapper">
       <!-- 启用二级缓存 -->
       <cache/>
       
       <select id="selectUserById" resultType="com.example.User">
           SELECT * FROM users WHERE id = #{id}
       </select>
   </mapper>
   ```

3. **在 `Mapper` 接口中不需要额外配置**，二级缓存会自动生效。

#### **二级缓存示例**

```
java复制编辑SqlSession session1 = sqlSessionFactory.openSession();
try {
    // 第一次查询，数据库执行查询操作
    User user1 = session1.selectOne("com.example.UserMapper.selectUserById", 1);

    // 关闭 session1，提交事务或手动提交缓存
    session1.close();

    // 重新打开一个新的 session，第二次查询，数据应该从二级缓存中获取
    SqlSession session2 = sqlSessionFactory.openSession();
    try {
        User user2 = session2.selectOne("com.example.UserMapper.selectUserById", 1);
        System.out.println(user1 == user2);  // 输出 true，表示从二级缓存中获取
    } finally {
        session2.close();
    }
} finally {
    session1.close();
}
```

#### **二级缓存失效情况**

- **SQL 语句发生变化**：如果查询条件发生变化，MyBatis 会自动清空二级缓存。
- **数据库数据更新**：当数据库中的数据发生变化（例如 `insert`、`update`、`delete`），二级缓存会失效，MyBatis 会将更新的数据重新写入缓存。
- **手动清除缓存**：可以通过 `session.clearCache()` 或其他 API 手动清除二级缓存。

### **二级缓存的配置与使用**

二级缓存的配置在 `Mapper` 的 XML 文件中进行。

1. **全局配置**：可以在 MyBatis 配置文件中启用全局缓存，设置全局缓存开关。

   ```
   xml复制编辑<settings>
       <setting name="cacheEnabled" value="true"/>
   </settings>
   ```

2. **`Mapper` 映射文件配置**：通过 `<cache/>` 元素在每个 `Mapper` 映射文件中开启缓存。

   ```
   xml复制编辑<mapper namespace="com.example.UserMapper">
       <cache/>
   </mapper>
   ```

3. **缓存的清理与更新**：

   - `flushInterval`：可以设置缓存刷新的时间间隔。
   - `size`：设置缓存的最大数量。
   - `eviction`：设置缓存的清理策略（如 FIFO、LRU）。
   - `readOnly`：设置缓存是否是只读的。

### **一级缓存与二级缓存的对比**

| 特性             | 一级缓存                                         | 二级缓存                                           |
| ---------------- | ------------------------------------------------ | -------------------------------------------------- |
| **作用范围**     | 单个 `SqlSession` 实例                           | 跨 `SqlSessionFactory` 的多个 `SqlSession`         |
| **生命周期**     | `SqlSession` 级别，`SqlSession` 关闭或提交后失效 | `SqlSessionFactory` 级别，跨 `SqlSession` 保持有效 |
| **是否自动启用** | 默认启用                                         | 需要手动启用                                       |
| **缓存数据存储** | 存储在 `SqlSession` 内存中                       | 存储在内存或其他持久化存储中                       |
| **缓存清空时机** | `SqlSession` 提交或关闭时清空                    | 根据配置或数据变化（如 `insert`、`update`）清空    |

### **总结**

- **一级缓存** 是 MyBatis 的默认缓存，作用范围是单个 `SqlSession`，缓存的内容在 `SqlSession` 提交或关闭时失效。
- **二级缓存** 是跨 `SqlSession` 的缓存，作用范围是 `SqlSessionFactory`，需要在配置文件中手动启用，并且支持多种存储策略（内存、文件系统等）。
- **一级缓存** 主要是为了提高同一个 `SqlSession` 内部的查询性能，而 **二级缓存** 则可以进一步提升跨 `SqlSession` 查询的性能，适用于长期不变的数据。



### **关闭 MyBatis 一级缓存的方法**

1. **通过配置文件禁用一级缓存**：

   在 MyBatis 配置文件（`mybatis-config.xml`）中，可以通过设置 `<settings>` 标签中的 `localCacheScope` 属性来控制一级缓存的启用与否。

   - ```
     localCacheScope
     ```

      可以设置为两种值：

     - `SESSION`（默认值）：启用一级缓存，`SqlSession` 生命周期内的所有查询都使用该缓存。
     - `STATEMENT`：禁用一级缓存，查询每次都会从数据库中获取数据。

   配置示例：

   ```
   xml复制编辑<configuration>
       <settings>
           <!-- 禁用一级缓存 -->
           <setting name="localCacheScope" value="STATEMENT"/>
       </settings>
   </configuration>
   ```

   **解释**：

   - `SESSION`：表示每个 `SqlSession` 在整个生命周期内都使用一级缓存。
   - `STATEMENT`：表示每次执行 SQL 语句时都不使用一级缓存，MyBatis 会每次查询都去数据库中获取数据。

2. **通过 `SqlSession` 手动清除一级缓存**：

   如果你不想完全禁用一级缓存，而是希望在某些情况下清空缓存，可以手动清空一级缓存。MyBatis 提供了 `clearCache()` 方法，用于在 `SqlSession` 内清空缓存。

   ```java
   java复制编辑SqlSession session = sqlSessionFactory.openSession();
   try {
       // 执行查询操作
       User user1 = session.selectOne("com.example.UserMapper.selectUserById", 1);
       
       // 手动清除一级缓存
       session.clearCache();
       
       // 再次查询，一级缓存已清空
       User user2 = session.selectOne("com.example.UserMapper.selectUserById", 1);
       
       System.out.println(user1 == user2);  // 输出 false，因为一级缓存已经被清除
   } finally {
       session.close();
   }
   ```

   在上述代码中，`session.clearCache()` 会清空当前 `SqlSession` 的一级缓存，导致下一次查询无法从缓存中获取数据。

### mybatis缓存扩展

1、实现org.apache.ibatis.cache.Cache接口，添加redis、ehcache等实现
2、在mapper.xml文件中声明所使用缓存实现的别名，默认是mybatis自己的实现PerpetualCache。
<cache value="redisCache"/>



源码

  SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);方法构建SqlSessionFactory时读取所有mapper.xml映射文件，将包含有<cache/ >标签解析出来，默认type=PERPETUAL，可修改type的值指定cache实现。

org.apache.ibatis.builder.xml.XMLConfigBuilder#parseConfiguration --》

org.apache.ibatis.builder.xml.XMLConfigBuilder#mapperElement --》

org.apache.ibatis.builder.xml.XMLMapperBuilder#configurationElement --》

**org.apache.ibatis.builder.xml.XMLMapperBuilder#cacheElement**    读取cache配置

```
String type = context.getStringAttribute("type", "PERPETUAL");
```

```java
private void configurationElement(XNode context) {
  try {
    String namespace = context.getStringAttribute("namespace");
    if (namespace == null || namespace.equals("")) {
      throw new BuilderException("Mapper's namespace cannot be empty");
    }
    builderAssistant.setCurrentNamespace(namespace);
    cacheRefElement(context.evalNode("cache-ref"));
    cacheElement(context.evalNode("cache"));
    parameterMapElement(context.evalNodes("/mapper/parameterMap"));
    resultMapElements(context.evalNodes("/mapper/resultMap"));
    sqlElement(context.evalNodes("/mapper/sql"));
    buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing Mapper XML. The XML location is '" + resource + "'. Cause: " + e, e);
  }
}

private void cacheElement(XNode context) throws Exception {
    if (context != null) {
      String type = context.getStringAttribute("type", "PERPETUAL");
      Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
      String eviction = context.getStringAttribute("eviction", "LRU");
      Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);
      Long flushInterval = context.getLongAttribute("flushInterval");
      Integer size = context.getIntAttribute("size");
      boolean readWrite = !context.getBooleanAttribute("readOnly", false);
      boolean blocking = context.getBooleanAttribute("blocking", false);
      Properties props = context.getChildrenAsProperties();
      builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, blocking, props);
    }
  }
```