### 1.6自定义bean

#### 1.61 生命周期回调

开发者要参与spring容器管理bean的生命周期，可以实现Spring的`InitializingBean` 和 `DisposableBean` 接口，实现afterPropertiesSet()、destroy()方法在初始化和销毁bean之后 完成自定义的动作。

##### 初始化回调

1. 实现InitializingBean接口，重写afterPropertiesSet()。不推荐这种方式因为它与Spring强关联

2. @PostConstruct，作用于初始化方法上

3. spring配置文件方式注入bean，使用init-method属性指定初始化方法

   ```xml
   <bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
   ```

   如果是在java config（java代码）中声明bean，可以使用@Bean指定初始化方法initMethod

```java
public class Foo {
    public void init() {
        // initialization logic
    }
}

public class Bar {
    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public Foo foo() {
        return new Foo();
    }

    @Bean(destroyMethod = "cleanup")
    public Bar bar() {
        return new Bar();
    }
}
```

三种方式比较

实现InitializingBean接口，速度快，但是与Spring强耦合

@PostConstruct注解，反射实现，速度慢，不与Spring关联

<bean>标签或者@Bean注解，反射实现，速度慢，不与



@PostConstruct通过注解反射获取初始化（init）方法

CommonAnnotationBeanPostProcessor
InitDestroyAnnotationBeanPostProcessor

```java
private LifecycleMetadata buildLifecycleMetadata(final Class<?> clazz) {
		List<LifecycleElement> initMethods = new ArrayList<>();
		List<LifecycleElement> destroyMethods = new ArrayList<>();
		Class<?> targetClass = clazz;

		do {
			final List<LifecycleElement> currInitMethods = new ArrayList<>();
			final List<LifecycleElement> currDestroyMethods = new ArrayList<>();

			ReflectionUtils.doWithLocalMethods(targetClass, method -> {
                  //反射获取初始化注解的方法
				if (this.initAnnotationType != null && method.isAnnotationPresent(this.initAnnotationType)) {
					LifecycleElement element = new LifecycleElement(method);
					currInitMethods.add(element);
					if (logger.isTraceEnabled()) {
						logger.trace("Found init method on class [" + clazz.getName() + "]: " + method);
					}
				}
                  //反射获取销毁注解的方法
				if (this.destroyAnnotationType != null && method.isAnnotationPresent(this.destroyAnnotationType)) {
					currDestroyMethods.add(new LifecycleElement(method));
					if (logger.isTraceEnabled()) {
						logger.trace("Found destroy method on class [" + clazz.getName() + "]: " + method);
					}
				}
			});

			initMethods.addAll(0, currInitMethods);
			destroyMethods.addAll(currDestroyMethods);
			targetClass = targetClass.getSuperclass();
		}
		while (targetClass != null && targetClass != Object.class);

		return new LifecycleMetadata(clazz, initMethods, destroyMethods);
	}
```



##### 销毁回调

1. 实现`DisposableBean` 的destroy() 方法。不推荐使用这种方式因为它与Spring是强关联

2. @PreDestory，作用于销毁回调方法

3. spring配置文件方式注入bean，使用destroy-method属性指定销毁回调方法

   ```xml
   <bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
   ```

```java
CommonAnnotationBeanPostProcessor
InitDestroyAnnotationBeanPostProcessor
```

##### 默认初始化、销毁方法

使用Spring xml <bean>或者java config注入bean，推荐申明方法为init()，destory()



##### 结合生命周期机制

三种初始化回调、销毁回调的方式，执行的先后顺序

Initialization methods

- Methods annotated with `@PostConstruct`

- `afterPropertiesSet()` as defined by the `InitializingBean` callback interface

- A custom configured `init()` method

  

Destroy methods are called in the same order:

- Methods annotated with `@PreDestroy`
- `destroy()` as defined by the `DisposableBean` callback interface
- A custom configured `destroy()` method



```java
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
            //初始化之前执行Bean的后置处理，CommonAnnotationBeanPostProcessor中会通过反射获取
            //Bean中有初始化注解的方法，反射调用该方法
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
            //调用Bean初始化方法(实现InitializingBean的afterPropertiesSet()方法)
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}
```



##### 开启关闭回调



#### 1.62 ApplicationContextAware and BeanNameAware



### 1.8用户扩展

