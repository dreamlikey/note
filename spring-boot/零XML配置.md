### **Spring vs Spring Boot 示例对比（零 XML 配置）**

以下对比展示了 **传统 Spring 应用** 与 **Spring Boot 应用** 在开发方式上的区别，重点体现在 **Spring Boot 的零 XML 配置** 和 **自动化能力**。

------

## **🔹 1. 传统 Spring 应用（使用 XML 配置）**

在 **Spring（非 Spring Boot）** 应用中，我们通常需要：

- **手动编写 `applicationContext.xml` 配置**
- **手动注册 `DispatcherServlet`**
- **需要外部 Web 服务器（如 Tomcat）**

### **📌 1.1 `applicationContext.xml` 配置**

```
xml复制编辑<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 定义一个 Service Bean -->
    <bean id="userService" class="com.example.service.UserService"/>
</beans>
```

------

### **📌 1.2 Java 代码**

#### **实体类**

```
java复制编辑public class User {
    private String name;
    public User(String name) { this.name = name; }
    public String getName() { return name; }
}
```

#### **Service 层**

```
java复制编辑public class UserService {
    public String getUser() {
        return "Hello, Spring User!";
    }
}
```

#### **Spring Main 方法**

```
java复制编辑import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringApp {
    public static void main(String[] args) {
        // 读取 XML 配置文件
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        
        // 获取 Bean
        UserService userService = context.getBean("userService", UserService.class);
        System.out.println(userService.getUser());
    }
}
```

📌 **缺点：**

- 需要手动维护 `applicationContext.xml`，不方便。
- 需要显式创建 `ApplicationContext`，配置较繁琐。

------

## **🔹 2. Spring Boot 应用（零 XML 配置）**

在 **Spring Boot** 中：

- **无须 `applicationContext.xml`**
- **无须手动注册 `DispatcherServlet`**
- **内置 Web 服务器，支持 `java -jar` 直接运行**
- **自动装配，简化配置**

### **📌 2.1 Spring Boot 代码**

#### **✅ `UserService`**

```
java复制编辑import org.springframework.stereotype.Service;

@Service  // 自动装配，无需 XML
public class UserService {
    public String getUser() {
        return "Hello, Spring Boot User!";
    }
}
```

#### **✅ Spring Boot 启动类**

```
java复制编辑import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication // 这个注解包含 @Configuration, @EnableAutoConfiguration, @ComponentScan
public class SpringBootApp {
    public static void main(String[] args) {
        // 直接运行 Spring Boot 应用
        ApplicationContext context = SpringApplication.run(SpringBootApp.class, args);
        
        // 获取 Bean
        UserService userService = context.getBean(UserService.class);
        System.out.println(userService.getUser());
    }
}
```

📌 **Spring Boot 直接运行，无需 XML，自动完成 Bean 扫描和配置。**

------

## **🔹 3. Spring vs Spring Boot 对比**

| 对比项         | 传统 Spring                         | Spring Boot                                          |
| -------------- | ----------------------------------- | ---------------------------------------------------- |
| **配置方式**   | 需要 XML (`applicationContext.xml`) | **零 XML**，基于注解                                 |
| **Web 服务器** | 需要手动部署到 Tomcat               | **内嵌 Tomcat/Jetty/Undertow**                       |
| **依赖管理**   | 需要手动引入多个 Spring 依赖        | **使用 `spring-boot-starter-xxx` 自动管理依赖**      |
| **Bean 配置**  | 需要手动声明 Bean                   | **自动扫描 `@Component`、`@Service`、`@Repository`** |
| **运行方式**   | 需要 Web 容器                       | **`java -jar` 直接运行**                             |

------

## **🔹 4. 结论**

✅ **Spring Boot 彻底摆脱 XML 配置，自动装配，简化了 Spring 开发流程。**
✅ **Spring Boot 更适合现代 Web 开发，支持微服务架构、快速部署。**
✅ **Spring 适用于复杂、大型系统的细粒度控制，但需要更多手动配置。**





### 传统 Spring也可以自动扫描bean吧





是的，**传统 Spring 也支持自动扫描 Bean**，但相比 Spring Boot 仍然有一些区别，主要体现在 **配置方式的繁琐程度** 和 **自动化程度** 上。

------

## **🔹 传统 Spring 的自动扫描 Bean**

Spring 早期使用 XML 配置 Bean（如 `<bean>` 标签），但从 Spring 2.5 开始，支持 **基于注解的 Bean 扫描**，可以用 `@ComponentScan` 代替 XML 配置。

### **📌 传统 Spring 自动扫描 Bean**

#### **1️⃣ 开启组件扫描**

在 `applicationContext.xml` 中启用 `component-scan`：

```
xml


复制编辑
<context:component-scan base-package="com.example.service"/>
```

#### **2️⃣ 标注 Spring 组件**

Spring 允许使用 `@Component`、`@Service`、`@Repository`、`@Controller` 自动注册 Bean：

```
java复制编辑import org.springframework.stereotype.Service;

@Service  // 让 Spring 自动管理这个 Bean
public class UserService {
    public String getUser() {
        return "Hello, Spring User!";
    }
}
```

#### **3️⃣ 运行 Spring 应用**

```java
java复制编辑import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class SpringApp {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        UserService userService = context.getBean(UserService.class);
        System.out.println(userService.getUser());
    }
}
```

📌 **虽然 Spring 也支持自动扫描 Bean，但仍然需要 `applicationContext.xml` 手动配置 `component-scan`，且需要手动创建 `ApplicationContext`**。

------

## **🔹 Spring Boot 自动扫描 Bean**

相比之下，Spring Boot **默认自动扫描 `@SpringBootApplication` 所在包及子包**，无需手动指定 `component-scan`。

### **📌 Spring Boot 代码**

```java
java复制编辑import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Service;
import org.springframework.context.ApplicationContext;

@SpringBootApplication  // 自动扫描 Bean
public class SpringBootApp {
    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(SpringBootApp.class, args);
        UserService userService = context.getBean(UserService.class);
        System.out.println(userService.getUser());
    }
}

@Service  // 直接生效，无需额外配置
class UserService {
    public String getUser() {
        return "Hello, Spring Boot User!";
    }
}
```

📌 **Spring Boot 省去了 `applicationContext.xml` 配置，并且默认扫描 `@SpringBootApplication` 所在包及其子包内的所有组件。**

------

## **🔹 关键区别总结**

| 特性                   | 传统 Spring                                   | Spring Boot                                         |
| ---------------------- | --------------------------------------------- | --------------------------------------------------- |
| **Bean 扫描**          | 需要手动配置 `<context:component-scan>`       | **自动扫描 `@SpringBootApplication` 包内所有 Bean** |
| **配置方式**           | 需要 `applicationContext.xml`                 | **完全基于 Java 注解，无需 XML**                    |
| **ApplicationContext** | 需要手动创建 `ClassPathXmlApplicationContext` | **Spring Boot 自动创建**                            |
| **额外步骤**           | 需要 XML 或 `@Configuration`                  | **零额外步骤，开箱即用**                            |

------

## **🔹 结论**

✅ **Spring（传统） 也支持自动扫描 Bean，但仍需手动配置 XML 或 `@ComponentScan`**。
✅ **Spring Boot 默认自动扫描，不需要 `@ComponentScan`，省去手动 XML 配置**。
✅ **Spring Boot 更适合现代开发，简化了 Spring 的复杂配置流程。**