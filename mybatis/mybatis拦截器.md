## 功能

mybatis拦截器作用是提供对mybatis功能的增强。

mybatis允许在映射语句执行过程中的某一点进行拦截，

允许开发者对：

Executor

ParameterHandler

ResultHandler

StatementHandler

这四个对象的相关方法实现增强

### 插件的核心功能

MyBatis 插件实现了 `Interceptor` 接口，拦截 MyBatis 的操作，并允许你在执行过程中修改行为。插件会对 `Executor`、`StatementHandler`、`ParameterHandler` 和 `ResultSetHandler` 进行拦截。

### 1. **Executor 插件**

`Executor` 是 MyBatis 执行 SQL 语句的核心对象，负责执行增、删、改、查操作。你可以通过插件拦截 `Executor` 的方法，来实现以下扩展：

- **SQL 执行监控**：记录 SQL 执行的日志，统计执行时间。
- **批量操作优化**：在执行批量操作时，插入自定义逻辑，优化批处理的方式。
- **缓存实现**：增强或替代 MyBatis 的二级缓存机制，执行自定义缓存策略。

### 2. **StatementHandler 插件**

`StatementHandler` 是 MyBatis 用来处理 SQL 语句的核心类，负责生成 `Statement` 对象并执行。插件可以拦截 `StatementHandler`，用于：

- **SQL 改写**：对 SQL 语句进行修改，如添加 SQL 日志、动态修改查询条件等。
- **SQL 打印**：记录 SQL 语句执行的详细信息，帮助调试。
- **SQL 优化**：根据某些条件对 SQL 进行优化，比如自动添加分页查询。

### 3. **ParameterHandler 插件**

`ParameterHandler` 用于设置 SQL 参数。通过插件，你可以：

- **参数加密**：在参数传递到数据库前，进行加密或格式化处理。
- **参数验证**：对传入 SQL 的参数进行验证，确保参数合法性。

### 4. **ResultSetHandler 插件**

`ResultSetHandler` 负责将 SQL 查询结果集 (`ResultSet`) 转换为 Java 对象。你可以通过插件：

- **结果集过滤或修改**：对查询结果进行过滤、修改或格式化。例如，给返回的对象添加额外的属性，或者对返回的数据进行统一的格式化。
- **结果集加解密**：对查询结果进行加密解密操作。

## 使用场景

1、拦截修改sql：分页插件PageHelper、分表查询【】

2、参数加密、参数验证、参数修改【特殊字符替换】

3、查询结果过滤、修改、加密

4、自定义缓存【增强或替代 MyBatis 的二级缓存机制，执行自定义缓存策略。】



## 实现

#### 插件实现示例

假设我们想要实现一个简单的性能监控插件，统计 SQL 执行时间：

```java
java复制编辑@Intercepts({
    @Signature(type = StatementHandler.class, method = "prepare", args = {Connection.class, Integer.class}),
    @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class}),
    @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})
})
public class PerformanceInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        long start = System.currentTimeMillis();

        // 执行目标方法
        Object result = invocation.proceed();

        long end = System.currentTimeMillis();
        System.out.println("SQL 执行时间: " + (end - start) + "ms");

        return result;
    }

    @Override
    public Object plugin(Object target) {
        // 为目标对象创建代理
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
        // 可以读取配置文件中的属性
    }
}
```

在这个例子中，我们创建了一个性能监控插件，拦截 `StatementHandler` 和 `Executor` 的 `prepare` 和 `query` 方法，在执行 SQL 前后统计执行时间。

#### 注册插件

要使插件生效，必须在 MyBatis 配置中进行注册。你可以在 `mybatis-config.xml` 中加入插件配置：

```xml
<plugins>
    <plugin interceptor="com.example.interceptor.PerformanceInterceptor"/>
</plugins>
```

或者通过 Java 配置：

```java
@Configuration
@MapperScan("com.example.mapper")
public class MyBatisConfig {

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);

        // 注册插件
        PerformanceInterceptor performanceInterceptor = new PerformanceInterceptor();
        factoryBean.setPlugins(new Interceptor[]{performanceInterceptor});

        return factoryBean.getObject();
    }

    @Bean
    public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

### 

