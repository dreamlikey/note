## **FactoryBean 的作用**

### **1. 什么是 `FactoryBean`？**

`FactoryBean<T>` 是 Spring 提供的一个**工厂 Bean**，用于**自定义 Bean 的创建逻辑**。

- **普通 Bean**：直接由 Spring 容器创建和管理。
- **`FactoryBean`**：本身是一个 Spring Bean，但它的 **`getObject()` 方法** 会**返回另一个对象**，Spring 容器最终管理的是 `FactoryBean` 生成的对象，而不是 `FactoryBean` 本身。

------

### **2. `FactoryBean` 的主要作用**

✅ **用来创建复杂对象，避免繁琐的 Bean 配置**（如数据库连接池、缓存对象）。
✅ **动态生成 Bean**，比如代理对象（用于 AOP）。
✅ **封装第三方库对象**，简化 Bean 注册。

------

### **3. `FactoryBean` 的核心方法**

`FactoryBean<T>` 是一个泛型接口，主要有 3 个方法：

```
java复制编辑public interface FactoryBean<T> {
    
    // 1. 返回实际要创建的 Bean
    T getObject() throws Exception;

    // 2. 返回创建的 Bean 的类型
    Class<?> getObjectType();

    // 3. 是否单例（默认 true）
    default boolean isSingleton() {
        return true;
    }
}
```

------

## **4. `FactoryBean` 使用案例**

### **案例 1：使用 `FactoryBean` 创建自定义对象**

假设我们有一个 `Car` 类：

```java
java复制编辑public class Car {
    private String brand;

    public Car(String brand) {
        this.brand = brand;
    }

    public String getBrand() {
        return brand;
    }
}
```

#### **创建 `CarFactoryBean` 来动态创建 `Car` 对象**

```java
java复制编辑import org.springframework.beans.factory.FactoryBean;
import org.springframework.stereotype.Component;

@Component
public class CarFactoryBean implements FactoryBean<Car> {

    @Override
    public Car getObject() {
        return new Car("Tesla"); // 这里可以是动态生成的对象
    }

    @Override
    public Class<?> getObjectType() {
        return Car.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

#### **在 Spring 里使用 `FactoryBean`**

```java
java复制编辑import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class CarService {

    @Autowired
    private Car car; // 这里注入的是 Car，而不是 CarFactoryBean

    public void printCar() {
        System.out.println("Car brand: " + car.getBrand());
    }
}
```

**输出：**

```
yaml


复制编辑
Car brand: Tesla
```

🔹 **解析：**

- `CarFactoryBean` **本身是一个 Spring Bean**，但 `getObject()` 返回的是 `Car`。
- `@Autowired` 注入的 **是 `Car`，而不是 `CarFactoryBean`**。

------

### **5. 获取 `FactoryBean` 本身**

如果需要获取 `FactoryBean` 实例，而不是它生成的对象，Spring 提供了 **`&` 前缀**：

```java
java复制编辑@Autowired
private CarFactoryBean carFactoryBean;  // 获取 FactoryBean 生成的 Car 对象

@Autowired
private FactoryBean<Car> factoryBean;   // 另一种方式，直接获取 FactoryBean

@Autowired
private Object factoryBeanObject;

@Autowired
private Object carBean;

public void printBeans() {
    System.out.println("factoryBeanObject class: " + factoryBeanObject.getClass());
    System.out.println("carBean class: " + carBean.getClass());
}
```

**如果手动获取 `FactoryBean` 本身：**

```
java


复制编辑
CarFactoryBean factoryBean = (CarFactoryBean) context.getBean("&carFactoryBean");
```

🔹 **解析：**

- `context.getBean("carFactoryBean")` 返回的是 `Car`。
- `context.getBean("&carFactoryBean")` 返回的是 `CarFactoryBean`。

------

### **6. `FactoryBean` 的应用场景**

#### **(1) 用于封装第三方库**

比如 `SqlSessionFactoryBean`（MyBatis），封装了 MyBatis 的 `SqlSessionFactory` 对象：

```java
java复制编辑@Bean
public SqlSessionFactoryBean sqlSessionFactory() {
    SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
    factoryBean.setDataSource(dataSource());
    return factoryBean;
}
```

#### **(2) 用于创建代理对象（AOP 代理）**

Spring AOP 使用 `ProxyFactoryBean` 动态生成代理对象：

```java
java复制编辑@Bean
public ProxyFactoryBean proxyFactoryBean() {
    ProxyFactoryBean proxyFactoryBean = new ProxyFactoryBean();
    proxyFactoryBean.setTarget(myService());
    proxyFactoryBean.addAdvice(new MyMethodInterceptor());
    return proxyFactoryBean;
}
```

#### **(3) 适用于需要动态创建 Bean 的情况**

比如不同环境加载不同的 Bean，可以通过 `FactoryBean` 实现。

------

## **7. `FactoryBean` vs `BeanFactoryPostProcessor`**

| 对比项              | `FactoryBean`                              | `BeanFactoryPostProcessor`               |
| ------------------- | ------------------------------------------ | ---------------------------------------- |
| 作用                | **用于创建 Bean**                          | **修改 Bean 的定义（`BeanDefinition`）** |
| 影响对象            | 只影响自己创建的 Bean                      | 影响整个 Spring 容器                     |
| 是否能动态生成 Bean | ✅ 可以                                     | ❌ 不能                                   |
| 常见应用            | **动态代理、第三方库封装、动态 Bean 生成** | **修改 `scope`、加载配置**               |

------

## **8. 总结**

✅ `FactoryBean` **本身是一个 Bean**，但最终返回的是 **`getObject()` 生成的 Bean**。
✅ 适用于 **创建复杂对象**，如代理对象、第三方库封装等。
✅ 通过 **`&beanName` 可以获取 `FactoryBean` 本身**，而不是它创建的对象。

🚀 **最佳实践**：

- **如果只是修改 Bean 的属性，使用 `BeanFactoryPostProcessor`**。
- **如果要创建动态 Bean，使用 `FactoryBean`**（如代理对象、数据库连接池）。
- **如果要注入 FactoryBean 本身，而不是它创建的对象，使用 `&beanName`**。

🎯 **一句话总结：** **`FactoryBean` 是 Spring 中用于** **"创建 Bean 的工厂"**，它允许我们**自定义对象创建逻辑**，用于**动态代理、封装第三方库、简化复杂对象注册**！