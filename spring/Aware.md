## **Spring 中 Aware 接口详解**

Spring **`Aware`** 接口用于让 **Bean 感知 Spring 容器中的特定对象或环境**，例如 `ApplicationContext`、`BeanFactory`、`ServletContext` 等。
它是一种 **回调机制**，当 Spring 创建 Bean 时，会自动调用 Aware 接口的方法，将特定对象注入到 Bean 中。

------

## **📌 1. Aware 接口作用**

- **让 Bean 获取 Spring 容器的内部对象**（如 `ApplicationContext`）。
- **帮助 Bean 访问容器资源**（如 `Environment`、`ResourceLoader`）。
- **增强 Bean 的功能**（如 `BeanNameAware` 获取 Bean 名称）。

**🌟 Aware 机制的本质**

- **Aware 是一个标记接口**，所有子接口都是 **单一方法的回调接口**。
- **Spring 会在 Bean 初始化时，自动调用这些 Aware 接口的方法**。

------

## **📌 2. 常见 Aware 接口**

| **接口**                         | **作用**                                       |
| -------------------------------- | ---------------------------------------------- |
| `BeanNameAware`                  | 获取当前 Bean 的名称                           |
| `BeanFactoryAware`               | 获取 `BeanFactory` 容器                        |
| `ApplicationContextAware`        | 获取 `ApplicationContext`                      |
| `EnvironmentAware`               | 获取 `Environment`（环境配置）                 |
| `ResourceLoaderAware`            | 获取 `ResourceLoader`（资源加载器）            |
| `MessageSourceAware`             | 获取 `MessageSource`（国际化消息）             |
| `ApplicationEventPublisherAware` | 获取 `ApplicationEventPublisher`（事件发布器） |

------

## **📌 3. Aware 接口使用示例**

### **🌟 示例 1：获取 `ApplicationContext`**

我们可以实现 `ApplicationContextAware` 来访问 Spring 容器。

```
java复制编辑package com.example.aware;

import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class MyApplicationContextAware implements ApplicationContextAware {
    private static ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        context = applicationContext;
        System.out.println("🔹 ApplicationContext 已注入: " + applicationContext);
    }

    public static ApplicationContext getContext() {
        return context;
    }
}
```

**💡 作用**：

- 让 `MyApplicationContextAware` 这个 Bean 获取 `ApplicationContext`。
- 其他地方可以通过 `MyApplicationContextAware.getContext()` 获取 Spring 容器。

------

### **🌟 示例 2：获取当前 Bean 名称**

`BeanNameAware` 让 Bean 访问自身的名称。

```
java复制编辑package com.example.aware;

import org.springframework.beans.factory.BeanNameAware;
import org.springframework.stereotype.Component;

@Component
public class MyBean implements BeanNameAware {
    private String beanName;

    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("🔹 当前 Bean 名称: " + name);
    }

    public String getBeanName() {
        return beanName;
    }
}
```

**💡 作用**：

- Spring 在创建 `MyBean` 时，会调用 `setBeanName()`，注入 Bean 名称。
- 方便 Bean 在运行时获取自身的名称。

------

### **🌟 示例 3：获取 BeanFactory**

`BeanFactoryAware` 让 Bean 访问 `BeanFactory`，可以手动获取其他 Bean。

```
java复制编辑package com.example.aware;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.stereotype.Component;

@Component
public class MyBeanFactoryAware implements BeanFactoryAware {
    private static BeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory factory) throws BeansException {
        beanFactory = factory;
        System.out.println("🔹 BeanFactory 已注入");
    }

    public static Object getBean(String name) {
        return beanFactory.getBean(name);
    }
}
```

**💡 作用**：

- 让 Bean 访问 `BeanFactory`，可以手动获取 Bean，而不依赖 `@Autowired`。

------

### **🌟 示例 4：访问环境变量**

`EnvironmentAware` 让 Bean 获取 `Environment`，可以访问应用配置。

```
java复制编辑package com.example.aware;

import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;
import org.springframework.core.env.EnvironmentAware;

@Configuration
public class MyEnvironmentAware implements EnvironmentAware {
    private static Environment environment;

    @Override
    public void setEnvironment(Environment env) {
        environment = env;
        System.out.println("🔹 Environment 已注入");
    }

    public static String getProperty(String key) {
        return environment.getProperty(key);
    }
}
```

**💡 作用**：

- 可以通过 `MyEnvironmentAware.getProperty("server.port")` 读取配置。

------

## **📌 4. Aware 机制的底层原理**

### **🌟 1. Spring 处理 Aware 的流程**

1. Spring **实例化 Bean**。
2. **检测是否实现 Aware 接口**。
3. **调用 Aware 方法**（如 `setApplicationContext()`）。
4. **完成 Bean 依赖注入**。

### **🌟 2. 关键源码**

Spring 通过 **`ApplicationContextAwareProcessor`** 处理 Aware 逻辑：

```
java复制编辑public void postProcessBeforeInitialization(Object bean, String beanName) {
    if (bean instanceof ApplicationContextAware) {
        ((ApplicationContextAware) bean).setApplicationContext(applicationContext);
    }
}
```

------

## **📌 5. Aware 的实际应用场景**

### **✅ 1. 解决循环依赖**

`ApplicationContextAware` 可以手动获取 Bean，避免 `@Autowired` 的循环依赖问题。

### **✅ 2. 统一管理 Spring 容器**

`ApplicationContextAware` 让我们在全局管理 `ApplicationContext`，其他类可以随时访问。

### **✅ 3. 访问环境变量**

`EnvironmentAware` 让 Bean 访问 `application.properties`，动态加载配置。

### **✅ 4. 资源加载**

`ResourceLoaderAware` 让 Bean 加载外部资源（如 XML、JSON）。

------

## **📌 6. 结论**

- **Aware 机制是 Spring 提供的一种回调机制**，可以让 Bean 感知 Spring 容器中的各种组件。
- **主要用于增强 Bean 功能**，如访问 `ApplicationContext`、`BeanFactory`、`Environment` 等。
- **Spring 自动调用 Aware 方法**，开发者只需要实现相应接口即可。
- **常见场景包括 Bean 访问环境变量、获取 Spring 容器、解决循环依赖等**。

------

✅ **总结：Spring Aware 让 Bean 感知 Spring 环境，提升应用灵活性，是 Spring 内部扩展的重要机制！** 🚀





### Aware在Bean生命周期中哪个阶段赋值的

在bean初始化过程中，为自定义属性赋值后调用initMethod之前对Aware接口的实现进行赋值。

具体分为两部分，

1、首先对BeanFactoryAware，BeanNameAware, ClassLoaderAware赋值

2、在BeanPostProcessor增强器的前置处理方法中调用ApplicationContextAwareProcessor#postProcessBeforeInitialization

ApplicationContextAwareProcessor是专门用于扩展Aware的增强器

```java
// 对应的Aware接口实现，进行相关容器属性赋值操作
@Override
@Nullable
public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
   AccessControlContext acc = null;

   if (System.getSecurityManager() != null &&
         (bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
               bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
               bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)) {
      acc = this.applicationContext.getBeanFactory().getAccessControlContext();
   }

   if (acc != null) {
      AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
         invokeAwareInterfaces(bean);
         return null;
      }, acc);
   }
   else {
      invokeAwareInterfaces(bean);
   }

   return bean;
}

// 对应的Aware接口实现，进行相关容器属性赋值操作
private void invokeAwareInterfaces(Object bean) {
   if (bean instanceof Aware) {
      if (bean instanceof EnvironmentAware) {
         ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
      }
      if (bean instanceof EmbeddedValueResolverAware) {
         ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
      }
      if (bean instanceof ResourceLoaderAware) {
         ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
      }
      if (bean instanceof ApplicationEventPublisherAware) {
         ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
      }
      if (bean instanceof MessageSourceAware) {
         ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
      }
      if (bean instanceof ApplicationContextAware) {
         ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
      }
   }
}
```

