#### 查询数据库示列

```java
public static void main(String[] args) {
  //1、Properties
  Properties properties = new Properties();
  properties.setProperty("driver","com.mysql.cj.jdbc.Driver");
  properties.setProperty("url","jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&serverTimezone=UTC");
  properties.setProperty("username","root");
  properties.setProperty("password","123456");

  //2、DataSource
  DataSourceFactory dataSourceFactory = new PooledDataSourceFactory();
  dataSourceFactory.setProperties(properties);
  DataSource dataSource = dataSourceFactory.getDataSource();

  //3、Transaction
  TransactionFactory transactionFactory = new JdbcTransactionFactory();

  //4、Environment
  Environment environment = new Environment("development", transactionFactory, dataSource);

  //5、Configuration
  Configuration configuration = new Configuration(environment);
  configuration.addMapper(UserMapper.class);

  //6、SqlSessionFactory
  SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
  SqlSession sqlSession = sqlSessionFactory.openSession();
  User user = sqlSession.selectOne("getByPK",1L);

  System.out.println(user.toString());

}
```

1、声明定义数据库连接属性 driver、url、username、password

2、通过数据库属性构建数据源（DataSource）

3、声明事务工厂TransactionFactory

4、声明mybatis环境Environment

5、声明mybatis配置Configuration，包含有数据源和事务工厂

6、通过Configuration，声明会话工厂SqlSessionFactory，生产与数据库的会话sqlSession

最后通过sqlSession，操作数据库





#### 环境以及sql的初始化过程



##### 创建sqlSession

数据库sql会话

**org.apache.ibatis.session.defaults.DefaultSqlSessionFactory#openSessionFromDataSource**

```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
  Transaction tx = null;
  try {
    final Environment environment = configuration.getEnvironment();
    //事务工厂
    final TransactionFactory transactionFactory = 	getTransactionFactoryFromEnvironment(environment);
      //创建当前会话事务
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
      //创建一个sql执行器，包含有用于当前会话的配置、事务、查询缓存（二级缓存）
    final Executor executor = configuration.newExecutor(tx, execType);
    return new DefaultSqlSession(configuration, executor, autoCommit);
  }  finally {
    ErrorContext.instance().reset();
  }
}
```



#### sql的执行过程

User user = sqlSession.selectOne("getByPK",1L);

statement 也就是sql的唯一标识，statement = namespace + id（或者 methodName）

**selectOne(String statement, Object parameter)**

通过statement(sql的唯一标识)找到已加载的sql声明MappedStatement（mybatis初始化过程中通过配置或指定的文件找到Mapper中的sql并保存到一个HashMap中，mapper.xml中每个sql的id做为key，value=MappedStatement）



```java
org.apache.ibatis.session.defaults.DefaultSqlSession

//statement 也就是sql的唯一标识
public <T> T selectOne(String statement, Object parameter) {
  // Popular vote was to return null on 0 results and throw exception on too many.
  List<T> list = this.selectList(statement, parameter);
  if (list.size() == 1) {
    return list.get(0);
  } 
}
@Override
  public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
    try {
      //获取匹配的Statement（声明），MappedStatement在初始化mybatis时加载到Configuration
      MappedStatement ms = configuration.getMappedStatement(statement);
      //执行器执行sql
      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
    } 
  }
```



**org.apache.ibatis.mapping.MappedStatement**

通过sql唯一标识找到mybatis初始化的具体MappedStatement(sql声明)，获取sql并拼接传入的参数parameter

```java
//获取所要执行的sql
public BoundSql getBoundSql(Object parameterObject) {
    
  BoundSql boundSql = sqlSource.getBoundSql(parameterObject);
  List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
  if (parameterMappings == null || parameterMappings.isEmpty()) {
    boundSql = new BoundSql(configuration, boundSql.getSql(), parameterMap.getParameterMappings(), parameterObject);
  }

  //拼接传入的sql参数parameter
  for (ParameterMapping pm : boundSql.getParameterMappings()) {
    String rmId = pm.getResultMapId();
    if (rmId != null) {
      ResultMap rm = configuration.getResultMap(rmId);
      if (rm != null) {
        hasNestedResultMaps |= rm.hasNestedResultMaps();
      }
    }
  }
  return boundSql;
}
```



**org.apache.ibatis.executor.Executor**

![1577168062248](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1577168062248.png)

```java
@Override
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
  //拿到sql	
  BoundSql boundSql = ms.getBoundSql(parameterObject);
  CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
  //运行sql
  return query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```



**org.apache.ibatis.executor.BaseExecutor#query(）**

```java
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
  if (closed) {
    throw new ExecutorException("Executor was closed.");
  }
  if (queryStack == 0 && ms.isFlushCacheRequired()) {
    clearLocalCache();
  }
  List<E> list;
  try {
    queryStack++;
    list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
    if (list != null) {
      handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
    } else {
      //执行sql
      list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
    }
  } finally {
    queryStack--;
  }
  if (queryStack == 0) {
    for (DeferredLoad deferredLoad : deferredLoads) {
      deferredLoad.load();
    }
    deferredLoads.clear();
    if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
      clearLocalCache();
    }
  }
  return list;
}


private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    List<E> list;
    localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
      //具体的执行sql逻辑
      //由具体的Excuter子类执行
      list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
      localCache.removeObject(key);
    }
    localCache.putObject(key, list);
    if (ms.getStatementType() == StatementType.CALLABLE) {
      localOutputParameterCache.putObject(key, parameter);
    }
    return list;
  }
```



**org.apache.ibatis.executor.SimpleExecutor#doQuery**

```java
@Override
public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
  Statement stmt = null;
  try {
    //mybatis配置
    Configuration configuration = ms.getConfiguration();
    //StatementHandler 创建一个处理器
    StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
    //预编译	
    stmt = prepareStatement(handler, ms.getStatementLog());
    //交给处理器去执行sql
    return handler.query(stmt, resultHandler);
  } finally {
    closeStatement(stmt);
  }
}
//预编译
private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    //数据库连接
    Connection connection = getConnection(statementLog);
    //通过处理器对sql进行预编译
    stmt = handler.prepare(connection, transaction.getTimeout());
    handler.parameterize(stmt);
    return stmt;
  }
```



**org.apache.ibatis.executor.BaseExecutor#getConnection**

```java
protected Connection getConnection(Log statementLog) throws SQLException {
  //transaction决定了以什么协议去连接数据库（jdbc或其它）在mybatis初始化的配置声明了	      TransactionFactory
  Connection connection = transaction.getConnection();
  if (statementLog.isDebugEnabled()) {
    return ConnectionLogger.newInstance(connection, statementLog, queryStack);
  } else {
    return connection;
  }
}
```



![1577170666322](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1577170666322.png)

**org.apache.ibatis.executor.statement.SimpleStatementHandler#query**

```java
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
  String sql = boundSql.getSql();
  //交给最终的数据库api去连接数据库并执行sql(例如jdbc)
  statement.execute(sql);
  return resultSetHandler.handleResultSets(statement);
}
```

这里测试使用了jdbc最终还是由jdbc去执行sql



#### sql注入



select * from user where name = ${name}

**$**符标识sql查询参数时，mybatis通过DynamicSqlSource动态sql去解析sql，

最终在底层SqlNode中通过StringBuilder.append(parameter)，将查询参数拼接在sql字符串上，造成sql注入

**类似于‘select * from user where name =’+name**

![1577174440056](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1577174440056.png)



select * from user where name = #{name}

**#**符标识sql查询参数时，不会发生sql注入，mybatis通过StaticSqlSource去解析sql，将参数作为参数对象传入

![1577175027395](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1577175027395.png)

最终执行的sql，避免出现sql注入

**select * from user where name = “'wudq' or 1=1”**



网上有人说是因为预编译的原因，是错误的，不论哪种情况都要





#### 缓存

##### 一级缓存

###### 定义

SqlSession级别缓存，同一个session多次执行相同查询，第二次及以后的查询会从缓存中取

SqlSessionFactory创建会话时创建的Excutor（BaseExecutor）中包含有PerpetualCache( localCache)实例，它包含一个HashMap成员变量用于存储查询结果，执行sql查询时将返回结果作为value保存到map中，下一次如果执行相同的查询（key相同）会从缓存中取。



一级缓存默认开启，默认作用域SESSION，若要关闭一级缓存作用域设为STATEMENT

```java
public enum LocalCacheScope {
  SESSION,STATEMENT
}
```

###### 源码

**org.apache.ibatis.executor.BaseExecutor**

```java
protected PerpetualCache localCache;

private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  List<E> list;
  localCache.putObject(key, EXECUTION_PLACEHOLDER);
  try {
    list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
  } finally {
    localCache.removeObject(key);
  }
  //查询结果存入一级缓存（HashMap）
  localCache.putObject(key, list);
  if (ms.getStatementType() == StatementType.CALLABLE) {
    localOutputParameterCache.putObject(key, parameter);
  }
  return list;
}

//删除部分代码
 public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  
    List<E> list;
    try {
      queryStack++;
      //是否存在一级缓存中
      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
      if (list != null) {
        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
      } else {
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
      }
    }
    return list;
  }
```

###### 使用

默认开启



##### 二级缓存

###### 定义

**二级缓存是Mapper级别的缓存**



mybatis初始化过程中，扫描映射文件，每个Mapper文件生成一个对应的MappedStatement实例存到Configuration实例的一个HashMap中，Mybatis通过Configuration构建SqlSessionFactory然后产生SqlSession，所有的SqlSession都是从同一个Configuration实例获取MappedStatement，就是说同一个查询都具有相同MappedStatement实例，MappedStatement对象中的Cache对象作为这个sql的查询缓存。



###### 源码

**org.apache.ibatis.session.Configuration**

```java
//全局共享的MappedStatement Map
protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection")
    .conflictMessageProducer((savedValue, targetValue) ->
        ". please check " + savedValue.getResource() + " and " + targetValue.getResource());
protected boolean cacheEnabled = true;

/**
   * 创建执行器Executor<br/>
   * 默认Simple执行器，如果开启了二级缓存cacheEnabled = true，则创建CachingExecutor
   */
  public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    //是否开启二级缓存，默认true
    if (cacheEnabled) {
      executor = new CachingExecutor(executor);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
  }
```

org.apache.ibatis.executor.CachingExecutor#query()

```java
@Override
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
    throws SQLException {
  //二级缓存
  Cache cache = ms.getCache();
  if (cache != null) {
    flushCacheIfRequired(ms);
    if (ms.isUseCache() && resultHandler == null) {
      ensureNoOutParams(ms, boundSql);
      //查询二级缓存
      List<E> list = (List<E>) tcm.getObject(cache, key);
      if (list == null) {
        list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
        tcm.putObject(cache, key, list);
      }
      return list;
    }
  }
  return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

###### 使用

使用二级缓存

1、cacheEnabled = true

2、声明Mapper使用二级缓存，@CacheNamespace、<cache/>



a、@CacheNamespace

```java
@CacheNamespace
public interface UserMapper {

  @Select("select * from user where id = #{id}")
  User getByPK(Long id);

  @Select("select * from user where name = #{name}")
  User getByName(String name);

}
```

b、 <cache/>

```xml
<mapper namespace="...UserMapper">
    <cache/><!-- 加上该句即可，使用默认配置、还有另外一种方式，在后面写出 -->
    ...
</mapper>
```



在上面的代码中`tcm.putObject(cache, key, list);`这句代码是缓存了结果。但是实际上直到sqlsession提交，MyBatis才以序列化的形式保存到了一个Map（默认的缓存配置）中。



查询多于修改时使用二级缓存
在查询操作远远多于增删改操作的情况下可以使用二级缓存。因为任何增删改操作都将刷新二级缓存，对二级缓存的频繁刷新将降低系统性能。





-1119507391:1432194084:org.apache.ibatis.test.UserMapper.getByPK:0:2147483647:select * from user where id = ?:1:development



-1119507391:1432194084:org.apache.ibatis.test.UserMapper.getByPK:0:2147483647:select * from user where id = ?:1:development





