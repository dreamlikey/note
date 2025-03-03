Spring Boot 读取配置的方式有多种，主要包括 `application.properties` / `application.yml` 文件、`@Value` 注解、`@ConfigurationProperties`、`Environment` 对象、`@PropertySource`、`CommandLineRunner`、以及外部配置等。

------

## **1. 读取 `application.properties` 或 `application.yml`**

Spring Boot **默认加载** `src/main/resources/` 目录下的 `application.properties` 或 `application.yml` 作为配置文件。

**示例 (`application.properties`)**

```properties
server.port=8081
app.name=MySpringApp
```

**示例 (`application.yml`)**

```yaml
server:
  port: 8081
app:
  name: MySpringApp
```

------

## **2. 使用 `@Value` 注解**

适用于**读取单个配置项**。

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class ConfigService {
    
    @Value("${server.port}") 
    private int port;
    
    @Value("${app.name}") 
    private String appName;
    
    public void printConfig() {
        System.out.println("Port: " + port);
        System.out.println("App Name: " + appName);
    }
}
```

**优点**：

- 直接使用 `@Value`，简单直观。
- 适用于少量的单个值。

**缺点**：

- **不适合复杂对象**（如 `List`、`Map`）。
- **不能动态刷新**。

------

## **3. 使用 `@ConfigurationProperties`**

适用于**批量读取配置**，尤其是对象结构化配置。

### **1️⃣ 配置文件**

```yaml
app:
  name: MySpringApp
  version: 1.0.0
  authors:
    - Alice
    - Bob
```

### **2️⃣ Java 类**

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
@ConfigurationProperties(prefix = "app")
public class AppConfig {
    private String name;
    private String version;
    private List<String> authors;

    // Getters and Setters
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }

    public List<String> getAuthors() { return authors; }
    public void setAuthors(List<String> authors) { this.authors = authors; }
}
```

### **3️⃣ 使用**

```java
import org.springframework.stereotype.Service;

@Service
public class ConfigService {
    private final AppConfig appConfig;

    public ConfigService(AppConfig appConfig) {
        this.appConfig = appConfig;
    }

    public void printConfig() {
        System.out.println("App Name: " + appConfig.getName());
        System.out.println("Version: " + appConfig.getVersion());
        System.out.println("Authors: " + appConfig.getAuthors());
    }
}
```

**优点**：

- **支持批量读取**，适合复杂结构（如 `List`、`Map`）。
- **支持自动绑定**。
- **可以结合 `@RefreshScope` 实现动态刷新**（Spring Cloud Config）。

------

## **4. 通过 `Environment` 读取**

适用于**在代码中动态获取配置**。

```java
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Service;

@Service
public class EnvConfigService {

    private final Environment environment;

    public EnvConfigService(Environment environment) {
        this.environment = environment;
    }

    public void printConfig() {
        String port = environment.getProperty("server.port");
        String appName = environment.getProperty("app.name", "DefaultApp");
        System.out.println("Port: " + port);
        System.out.println("App Name: " + appName);
    }
}
```

**优点**：

- **可以动态获取**配置值。
- **适用于某些运行时需要动态读取的场景**。

**缺点**：

- **不够直观**，比 `@Value` 复杂。

------

## **5. 使用 `@PropertySource` 读取外部配置**

如果配置不在 `application.properties`，而是放在**其他文件**（如 `custom.properties`），可以用 `@PropertySource` 加载。

**1️⃣ 新建 `config/custom.properties`**

```properties
custom.key1=value1
custom.key2=value2
```

**2️⃣ Java 代码**

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;

@Configuration
@PropertySource("classpath:config/custom.properties")
public class CustomConfig {

    @Value("${custom.key1}")
    private String key1;

    public String getKey1() {
        return key1;
    }
}
```

**优点**：

- **可以加载自定义配置文件**，适用于非默认 `application.properties`。

**缺点**：

- **不能自动刷新**，如果文件更改，需要重启应用。

------

## **6. 通过 `CommandLineRunner` 读取**

适用于**在应用启动时读取并处理配置**。

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Component;

@Component
public class StartupConfig implements CommandLineRunner {

    private final Environment environment;

    public StartupConfig(Environment environment) {
        this.environment = environment;
    }

    @Override
    public void run(String... args) {
        System.out.println("App Name: " + environment.getProperty("app.name"));
    }
}
```

**优点**：

- **适用于启动时获取配置并执行初始化逻辑**。

------

## **7. 读取外部环境变量**

Spring Boot **支持环境变量**，可通过 `System.getenv()` 访问：

```java
System.out.println(System.getenv("JAVA_HOME"));
```

或者通过 `Environment` 读取：

```java
String dbHost = environment.getProperty("DB_HOST");
```

------

## **8. 通过命令行参数传入配置**

```shell
java -jar myapp.jar --server.port=9090 --app.name=CLIApp
```

Spring Boot **优先读取**命令行参数，覆盖 `application.properties`。

------

## **9. 读取远程配置（Spring Cloud Config）**

如果是 **Spring Cloud** 项目，可以从远程 `Config Server` 获取配置：

```properties
spring.cloud.config.uri=http://config-server:8888
```

------

## **10. 总结**

| **方式**                   | **适用场景**           | **优点**                     | **缺点**         |
| -------------------------- | ---------------------- | ---------------------------- | ---------------- |
| `@Value`                   | 读取**单个值**         | 直接读取，简单               | 不能读取复杂对象 |
| `@ConfigurationProperties` | **批量绑定对象**       | 适用于 `List`、`Map`，可刷新 | 需要创建 POJO 类 |
| `Environment`              | **动态读取配置**       | 运行时获取，可指定默认值     | 代码复杂度稍高   |
| `@PropertySource`          | **读取自定义配置文件** | 支持外部文件                 | 不能动态刷新     |
| `CommandLineRunner`        | **应用启动时读取配置** | 可执行初始化逻辑             | 仅适用于启动时   |
| **环境变量**               | **读取系统变量**       | 无需改代码                   | 需要设置环境变量 |
| **命令行参数**             | **启动时动态配置**     | 覆盖配置文件                 | 需手动传参       |
| **Spring Cloud Config**    | **远程读取配置**       | 适用于微服务架构             | 需要额外配置     |

------

🚀 **最佳实践**：

- **少量简单配置** → `@Value`
- **复杂结构（List/Map）** → `@ConfigurationProperties`
- **需要动态获取** → `Environment`
- **自定义配置文件** → `@PropertySource`
- **云环境 / 微服务** → `Spring Cloud Config`