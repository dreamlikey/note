这里是**Java Config** 方式的 Spring + MyBatis 集成，不再使用 `mybatis-config.xml`，所有 MyBatis 配置都写在 **Java 代码里**。

------

## ** Java Config 方式**

------

### **1️⃣ 依赖（pom.xml）**

```xml
xml复制编辑<dependencies>
    <!-- Spring 核心 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.3.30</version>
    </dependency>

    <!-- MyBatis Spring 适配 -->
    <dependency>
        <groupId>org.mybatis.spring</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.7</version>
    </dependency>

    <!-- MyBatis 核心 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.13</version>
    </dependency>

    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>

    <!-- HikariCP 数据库连接池 -->
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>5.0.1</version>
    </dependency>
</dependencies>
```

------

### **2️⃣ `SpringMyBatisConfig.java`（核心 Java 配置）**

```java
java复制编辑package com.example.config;

import com.zaxxer.hikari.HikariDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.Configuration;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
@ComponentScan(basePackages = "com.example")  // 扫描 Service 层
@MapperScan("com.example.mapper")  // 扫描 MyBatis Mapper 接口
public class SpringMyBatisConfig {

    // 1️⃣ 配置数据源（使用 HikariCP）
    @Bean
    public DataSource dataSource() {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/testdb?useSSL=false&serverTimezone=UTC");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        dataSource.setMaximumPoolSize(10);
        return dataSource;
    }

    // 2️⃣ 配置 SqlSessionFactory（取代 mybatis-config.xml）
    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource);

        // 配置 MyBatis 参数（Java 方式）
        org.apache.ibatis.session.Configuration configuration = new Configuration();
        configuration.setMapUnderscoreToCamelCase(true); // 开启驼峰命名匹配
        factoryBean.setConfiguration(configuration);

        return factoryBean.getObject();
    }
}
```

------

### **3️⃣ User 实体类**

```java
java复制编辑package com.example.model;

public class User {
    private int id;
    private String name;
    private int age;

    // Getters & Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}
```

------

### **4️⃣ UserMapper（MyBatis Mapper）**

```java
java复制编辑package com.example.mapper;

import com.example.model.User;
import org.apache.ibatis.annotations.*;

@Mapper
public interface UserMapper {

    @Select("SELECT * FROM user WHERE id = #{id}")
    User getUserById(@Param("id") int id);

    @Insert("INSERT INTO user (name, age) VALUES (#{name}, #{age})")
    void insertUser(User user);
}
```

------

### **5️⃣ UserService（业务逻辑层）**

```
java复制编辑package com.example.service;

import com.example.mapper.UserMapper;
import com.example.model.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public User getUserById(int id) {
        return userMapper.getUserById(id);
    }

    public void insertUser(User user) {
        userMapper.insertUser(user);
    }
}
```

------

### **6️⃣ 测试 MyBatis 配置**

```java
java复制编辑package com.example;

import com.example.config.SpringMyBatisConfig;
import com.example.model.User;
import com.example.service.UserService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class TestMyBatis {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(SpringMyBatisConfig.class);
        UserService userService = context.getBean(UserService.class);

        // 插入用户
        User newUser = new User();
        newUser.setName("张三");
        newUser.setAge(25);
        userService.insertUser(newUser);
        System.out.println("用户插入成功！");

        // 查询用户
        User user = userService.getUserById(1);
        System.out.println("查询到用户：" + user.getName() + ", 年龄：" + user.getAge());
    }
}
```