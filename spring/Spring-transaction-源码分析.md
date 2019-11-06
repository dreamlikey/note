##### 整体调用过程

###### 创建代理

Spring Ioc容器加载并初始化Bean的过程中，会对每个bean执行所有的后置处理（所有实现了BeanPostProcessor接口的类），AbstractAtuoProxyCreator也是其中一个，在它的后置处理中对当前Bean进行代理检查（是否需要创建代理类），如果需要则通过JDK动态代理或CGLIB生成代理类创建代理实例。

**Spring Ioc容器初始化Bean时通过AbstractAtuoProxyCreator的后置处理生成代理**

原生对象

![1571218163575](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1571218163575.png)

**经过后置处理过后，返回CGLIB代理对象（result）给IOC容器，这就是为什么获取的Spring Bean是一个代理对象的原因**，代理实现了事务处理但不仅限于事务，还有其它的功能增强

![1571218813535](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1571218813535.png)



**循环调用所有后置处理，AbstractAtuoProxyCreator包含在其中**

```java
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
      throws BeansException {

   Object result = existingBean;
   for (BeanPostProcessor processor : getBeanPostProcessors()) {
      Object current = processor.postProcessAfterInitialization(result, beanName);
      if (current == null) {
         return result;
      }
      result = current;
   }
   return result;
}
```

Spring Ioc容器最终

###### 代理添加事务增加



**debug调用追踪全图**

![1571215229958](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1571215229958.png)

##### 1-TransactionAutoConfiguration

spring-boot对Spring trancation的自动配置类，EnableTransactionManagementConfiguration是其中一个静态内部类，**启用事务管理配置**

```java
启用事务管理配置

@Configuration
@ConditionalOnBean(PlatformTransactionManager.class)
@ConditionalOnMissingBean(AbstractTransactionManagementConfiguration.class)
public static class EnableTransactionManagementConfiguration {

   @Configuration
   @EnableTransactionManagement(proxyTargetClass = false)
   @ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false", matchIfMissing = false)
   public static class JdkDynamicAutoProxyConfiguration {

   }

   @Configuration
   @EnableTransactionManagement(proxyTargetClass = true)
   @ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true", matchIfMissing = true)
   public static class CglibAutoProxyConfiguration {

   }

}
```

@EnableTransactionManagement  启用事务管理注解，它的作用是**启用Spring的注解驱动事务管理**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(TransactionManagementConfigurationSelector.class)
public @interface EnableTransactionManagement {
    //省略部分代码
}
```



##### 2-TransactionManagementConfigurationSelector

org.springframework.transaction.annotation.TransactionManagementConfigurationSelector事务管理配置选择器

```java
public class TransactionManagementConfigurationSelector extends AdviceModeImportSelector<EnableTransactionManagement> {

	/**
	 * {@inheritDoc}
	 * @return {@link ProxyTransactionManagementConfiguration} or
	 * {@code AspectJTransactionManagementConfiguration} for {@code PROXY} and
	 * {@code ASPECTJ} values of {@link EnableTransactionManagement#mode()}, respectively
	 */
	@Override
	protected String[] selectImports(AdviceMode adviceMode) {
		switch (adviceMode) {
			case PROXY:
				return new String[] {AutoProxyRegistrar.class.getName(), ProxyTransactionManagementConfiguration.class.getName()};
			case ASPECTJ:
				return new String[] {TransactionManagementConfigUtils.TRANSACTION_ASPECT_CONFIGURATION_CLASS_NAME};
			default:
				return null;
		}
	}

}
```

导入了AutoProxyRegistrar，ProxyTransactionManagementConfiguration两个类



##### 3-AutoProxyRegistrar

org.springframework.context.annotation.AutoProxyRegistrar 注册自动代理创建器



```java
注册Spring bean
public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		boolean candidateFound = false;
		Set<String> annoTypes = importingClassMetadata.getAnnotationTypes();
		for (String annoType : annoTypes) {
			AnnotationAttributes candidate = AnnotationConfigUtils.attributesFor(importingClassMetadata, annoType);
			if (candidate == null) {
				continue;
			}
			Object mode = candidate.get("mode");
			Object proxyTargetClass = candidate.get("proxyTargetClass");
			if (mode != null && proxyTargetClass != null && AdviceMode.class == mode.getClass() &&
					Boolean.class == proxyTargetClass.getClass()) {
				candidateFound = true;
				if (mode == AdviceMode.PROXY) {
					////注册自动代理创建器
					AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry);
					if ((Boolean) proxyTargetClass) {
						AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
						return;
					}
				}
			}
		}
    //省略部分代码
｝
```

进入registerAutoProxyCreatorIfNecessary(registry)方法，注册InfrastructureAdvisorAutoProxyCreator到IOC容器

```java
@Nullable
public static BeanDefinition registerAutoProxyCreatorIfNecessary(
      BeanDefinitionRegistry registry, @Nullable Object source) {
   //注册InfrastructureAdvisorAutoProxyCreator到IOC容器
   return registerOrEscalateApcAsRequired(InfrastructureAdvisorAutoProxyCreator.class, registry, source);
}
```

InfrastructureAdvisorAutoProxyCreator是AbstractAutoProxyCreator的子类，**AbstractAutoProxyCreator是通用代理创建器，作用是为声明了通知（Advice）的spring bean构建AOP代理**



###### AbstractAutoProxyCreator

AbstractAutoProxyCreator是通用代理创建器，为声明了通知（Advice）的spring bean构建AOP代理

AbstractAutoProxyCreator实现了BeanPostProcessor接口，也就是说在Spring Ioc容器初始化过程中对会对AbstractAutoProxyCreator进行后置处理

![1571128587873](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1571128587873.png)

实现了Spring bean后置处理的postProcessAfterInitialization()，postProcessBeforeInitialization()方法

```java
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName) {
   return bean;
}

/**
 * Create a proxy with the configured interceptors if the bean is
 * identified as one to proxy by the subclass.
 * @see #getAdvicesAndAdvisorsForBean
 */
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) throws BeansException {
   if (bean != null) {
      Object cacheKey = getCacheKey(bean.getClass(), beanName);
      if (!this.earlyProxyReferences.contains(cacheKey)) {
         //满足条件的进行代理包装
         return wrapIfNecessary(bean, beanName, cacheKey);
      }
   }
   return bean;
}
```



**wrapIfNecessary()方法对传入的bean进行了代理包装**

```java
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
   if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
      return bean;
   }
   if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
      return bean;
   }
   if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
      this.advisedBeans.put(cacheKey, Boolean.FALSE);
      return bean;
   }

   // Create proxy if we have advice.
   //获取bean的切面和通知
   Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
   if (specificInterceptors != DO_NOT_PROXY) {
      this.advisedBeans.put(cacheKey, Boolean.TRUE);
      Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
      this.proxyTypes.put(cacheKey, proxy.getClass());
      return proxy;
   }

   this.advisedBeans.put(cacheKey, Boolean.FALSE);
   return bean;
}
```

创建代理的过程

```java
	protected Object createProxy(Class<?> beanClass, @Nullable String beanName,
			@Nullable Object[] specificInterceptors, TargetSource targetSource) {

		if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
			AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
		}

		ProxyFactory proxyFactory = new ProxyFactory();
		proxyFactory.copyFrom(this);

		if (!proxyFactory.isProxyTargetClass()) {
			if (shouldProxyTargetClass(beanClass, beanName)) {
				proxyFactory.setProxyTargetClass(true);
			}
			else {
				evaluateProxyInterfaces(beanClass, proxyFactory);
			}
		}

		Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
		proxyFactory.addAdvisors(advisors);
		proxyFactory.setTargetSource(targetSource);
		customizeProxyFactory(proxyFactory);

		proxyFactory.setFrozen(this.freezeProxy);
		if (advisorsPreFiltered()) {
			proxyFactory.setPreFiltered(true);
		}

		return proxyFactory.getProxy(getProxyClassLoader());
	}

```



##### 4-ProxyTransactionManagementConfiguration

配置 注册启用基于代理的注解驱动事务管理所必需的Spring基础bean

```java
{@code @Configuration} class that registers the Spring infrastructure beans
necessary to enable proxy-based annotation-driven transaction management.
```



AOP代理包含有切面、通知、切入点，切面=通知+切入点

```java
@Configuration
public class ProxyTransactionManagementConfiguration extends AbstractTransactionManagementConfiguration {

   @Bean(name = TransactionManagementConfigUtils.TRANSACTION_ADVISOR_BEAN_NAME)
   @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
   public BeanFactoryTransactionAttributeSourceAdvisor transactionAdvisor() {
      BeanFactoryTransactionAttributeSourceAdvisor advisor = new BeanFactoryTransactionAttributeSourceAdvisor();
      advisor.setTransactionAttributeSource(transactionAttributeSource());
      advisor.setAdvice(transactionInterceptor());
      if (this.enableTx != null) {
         advisor.setOrder(this.enableTx.<Integer>getNumber("order"));
      }
      return advisor;
   }

   @Bean
   @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
   public TransactionAttributeSource transactionAttributeSource() {
      return new AnnotationTransactionAttributeSource();
   }

   @Bean
   @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
   public TransactionInterceptor transactionInterceptor() {
      TransactionInterceptor interceptor = new TransactionInterceptor();
      interceptor.setTransactionAttributeSource(transactionAttributeSource());
      if (this.txManager != null) {
         interceptor.setTransactionManager(this.txManager);
      }
      return interceptor;
   }

}
```

###### BeanFactoryTransactionAttributeSourceAdvisor

**功能**：依据TransactionAttributeSource类所提供的属性值驱动一个AOP通知，
用来将使用了@Transactional注解的方法加入到事务切面的bean中，也就是**通过事务属性构建事务代理类的pointcut、advice**

![1571122914975](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1571122914975.png)

BeanFactoryTransactionAttributeSourceAdvisor继承自AbstractBeanFactoryPointcutAdvisor，AbstractBeanFactoryPointcutAdvisor就是用于创建AOP切入点 的BeanFactory



BeanFactoryTransactionAttributeSourceAdvisor类中包含有TransactionAttributeSource、Pointcut、Advice



```java
通过事务属性构建事务代理类的pointcut、advice

public class BeanFactoryTransactionAttributeSourceAdvisor extends AbstractBeanFactoryPointcutAdvisor {

	@Nullable
	private TransactionAttributeSource transactionAttributeSource;

	private final TransactionAttributeSourcePointcut pointcut = new TransactionAttributeSourcePointcut() {
		@Override
		@Nullable
		protected TransactionAttributeSource getTransactionAttributeSource() {
			return transactionAttributeSource;
		}
	};

	public void setTransactionAttributeSource(TransactionAttributeSource transactionAttributeSource) {
		this.transactionAttributeSource = transactionAttributeSource;
	}

	@Override
	public Pointcut getPointcut() {
		return this.pointcut;
	}

}
```

![1571122872300](C:\Users\jiachao\AppData\Roaming\Typora\typora-user-images\1571122872300.png)

###### **TransactionAttributeSource**

声明一个事务AOP需要的属性

###### **TransactionInterceptor**

就是事务AOP的通知（Spring aop的文档中也表明Advice通常被定义为拦截器）



##### 5-TransactionInterceptor

##### 6-Advisor



##### 7-AopProxy

###### JdkDynamicAopProxy

```java
@Override
	public Object getProxy(@Nullable ClassLoader classLoader) {
		if (logger.isDebugEnabled()) {
			logger.debug("Creating JDK dynamic proxy: target source is " + this.advised.getTargetSource());
		}
		Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
		findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
		return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
	}

```

**JDK反射重新创建一个功能增加后的实例，这个实例就是代理类**

动态编译生成字节码，再反编译字节码生成代理类

JDK动态代理部分源码

```java
 public static Object newProxyInstance(ClassLoader loader,
                                       Class<?>[] interfaces,
                                       InvocationHandler h)
        throws IllegalArgumentException
    {
        Objects.requireNonNull(h);
        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * Look up or generate the designated proxy class.
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }

            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            return cons.newInstance(new Object[]{h});
        }
    }
```



###### CglibApoProxy

```java
private static class DynamicAdvisedInterceptor implements MethodInterceptor, Serializable {

   private final AdvisedSupport advised;

   public DynamicAdvisedInterceptor(AdvisedSupport advised) {
      this.advised = advised;
   }

   @Override
   @Nullable
   public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
      Object oldProxy = null;
      boolean setProxyContext = false;
      Object target = null;
      TargetSource targetSource = this.advised.getTargetSource();
      try {
         if (this.advised.exposeProxy) {
            // Make invocation available if necessary.
            oldProxy = AopContext.setCurrentProxy(proxy);
            setProxyContext = true;
         }
         // Get as late as possible to minimize the time we "own" the target, in case it comes from a pool...
         target = targetSource.getTarget();
         Class<?> targetClass = (target != null ? target.getClass() : null);
         //获取通知，此处的通知为TransactionInterceptor
         List<Object> chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method, targetClass);
         Object retVal;
         // Check whether we only have one InvokerInterceptor: that is,
         // no real advice, but just reflective invocation of the target.
         if (chain.isEmpty() && Modifier.isPublic(method.getModifiers())) {
            // We can skip creating a MethodInvocation: just invoke the target directly.
            // Note that the final invoker must be an InvokerInterceptor, so we know
            // it does nothing but a reflective operation on the target, and no hot
            // swapping or fancy proxying.
            Object[] argsToUse = AopProxyUtils.adaptArgumentsIfNecessary(method, args);
            retVal = methodProxy.invoke(target, argsToUse);
         }
         else {
            // We need to create a method invocation...
            retVal = new CglibMethodInvocation(proxy, target, method, args, targetClass, chain, methodProxy).proceed();
         }
         retVal = processReturnType(proxy, target, method, retVal);
         return retVal;
      }
      finally {
         if (target != null && !targetSource.isStatic()) {
            targetSource.releaseTarget(target);
         }
         if (setProxyContext) {
            // Restore old proxy.
            AopContext.setCurrentProxy(oldProxy);
         }
      }
   }
```

