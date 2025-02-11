### **Spring Boot 是什么？**

Spring Boot 是 **Spring 框架的一个子项目**，它用来**简化 Spring 应用的开发**，让开发者可以**快速构建独立运行的、生产级别的 Spring 应用**，而**无需繁琐的 XML 配置**。

------

### **📌 Spring Boot 的核心特点**

1. **零 XML 配置**：基于 Java 注解 (`@SpringBootApplication`) 实现自动配置。
2. **内嵌 Web 服务器**：默认支持 Tomcat、Jetty、Undertow，无需手动配置。
3. **自动装配（AutoConfiguration）**：自动根据依赖配置合适的 Bean。
4. **Starter 机制**：通过 `spring-boot-starter-xxx` 依赖，一键引入所需功能。
5. **开箱即用**：默认提供合理的配置，开发者可以根据需要修改。
6. **独立运行**：可以打包成 **JAR** 文件，直接使用 `java -jar` 运行，无需外部服务器。

------

### **📌 Spring Boot 的基本结构**

Spring Boot 应用的入口通常是一个 `@SpringBootApplication` 注解的类，例如：

```
java复制编辑import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

✅ **这个类做了三件事：**

- **`@SpringBootApplication`**：开启 Spring Boot 的自动配置和组件扫描。
- **`SpringApplication.run(...)`**：启动 Spring Boot 应用。
- **自动加载 `application.properties` 配置**，进行 Bean 装配。

------

### **📌 Spring Boot 主要组件**

| 组件                              | 作用                                                    |
| --------------------------------- | ------------------------------------------------------- |
| **Spring Boot Starter**           | 预定义的依赖集合，如 `spring-boot-starter-web`          |
| **Spring Boot AutoConfiguration** | 根据依赖自动配置 Bean，如 `DataSourceAutoConfiguration` |
| **Spring Boot CLI**               | 允许使用 Groovy 语法快速构建 Spring 应用                |
| **Spring Boot Actuator**          | 提供应用监控与管理功能                                  |
| **Spring Boot DevTools**          | 提供开发环境优化，如热部署                              |

------

### **📌 Spring Boot vs Spring**

| 特性       | Spring                | Spring Boot                    |
| ---------- | --------------------- | ------------------------------ |
| 配置方式   | 需要大量 XML 配置     | **零 XML，基于注解**           |
| Web 服务器 | 需要手动部署到 Tomcat | **自带 Tomcat/Jetty/Undertow** |
| 依赖管理   | 需要手动管理          | **提供 Starter 依赖**          |
| 运行方式   | 需要外部服务器        | **可打包成 JAR，直接运行**     |

------

### **📌 Spring Boot 适用于哪些场景？**

1. **快速开发 RESTful API**
2. **微服务架构（Spring Cloud）**
3. **企业级应用（如电商、金融、ERP 等）**
4. **云原生应用（Cloud-Native Apps）**
5. **独立运行的后台任务或脚本**

------

### **📌 结论**

✅ **Spring Boot = Spring + 自动配置 + 内嵌服务器 + 约定优于配置**
✅ **极大简化了 Spring 开发，让开发者专注于业务逻辑**
✅ **适合快速开发微服务、Web 应用、独立应用等** 🚀



SpringbootApplication.run()方法



![springboo启动流程](.\springboo启动流程.png)