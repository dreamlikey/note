#### 实现原理

<https://www.jianshu.com/p/8ff9201ed7d6>

##### CGLIB

```java
org.springframework.aop.framework.CglibAopProxy.DynamicAdvisedInterceptor#intercept
```



用 `TransactionProxyFactoryBean` 接口来使用 AOP 功能，生成 proxy 代理对象，通过 

TransactionInterceptor拦截到代理方法，将事务功能添加在 方法前后，通过代理调用方法。

***方法调用前***，初始化事务属性、事务管理器、切入点id、事务信息， 

**方法调用后**，执行事务的 回滚、提交、清空操作。

创建数据库连接带事务，同一个事务使用同一个数据库连接，不同事务创建不同数据库连接。



*org.springframework.transaction.interceptor.TransactionAspectSupport#invokeWithinTransaction*

```java
protected Object invokeWithinTransaction(Method method, @Nullable Class<?> targetClass,
			final InvocationCallback invocation) throws Throwable {

		// If the transaction attribute is null, the method is non-transactional.
		TransactionAttributeSource tas = getTransactionAttributeSource();
		final TransactionAttribute txAttr = (tas != null ? tas.getTransactionAttribute(method, targetClass) : null);
		final PlatformTransactionManager tm = determineTransactionManager(txAttr);
		final String joinpointIdentification = methodIdentification(method, targetClass, txAttr);

		if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {
			// Standard transaction demarcation with getTransaction and commit/rollback calls.
			TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
			Object retVal = null;
			try {
				// This is an around advice: Invoke the next interceptor in the chain.
				// This will normally result in a target object being invoked.
				retVal = invocation.proceedWithInvocation();
			} catch (Throwable ex) {
				// target invocation exception
                //回滚事务
				completeTransactionAfterThrowing(txInfo, ex);
				throw ex;
			} finally {
                //清空当前事务信息
				cleanupTransactionInfo(txInfo);
			}
            //提交事务
			commitTransactionAfterReturning(txInfo);
			return retVal;
		}

		else {
			//省略部分代码
			}
		}
	}
```





```java
org.springframework.aop.framework.CglibAopProxy.intercept()
    org.springframework.aop.framework.ReflectiveMethodInvocation.proceed()
		TransactionInterceptor.invoke()
    		TransactionAspectSupport.invokeWithinTransaction()

```

```java
invokeWithinTransaction(Method method, @Nullable Class<?> targetClass,
      final InvocationCallback invocation)
```







#### 事务的隔离级别

Isolation



##### default 

​	默认数据库隔离级别

##### read_commited  读提交

​	读取不受限制，最低的隔离级别，会引起脏读、幻读、不可重复读问题

##### read_uncommited 读未提交

​	读取受限，保证一个事务数据提交之后其它事务才能读取，解决脏读问题

##### repeatable_read 可重复读

​	防止脏读、不可重复读，会出现幻读

​	不可重复读：同一个事务两次读取同一条数据，不一致

​	因为其他事务对这条数据进行了提交

##### serializable 串行化

​	花费最高的事务隔离级别，事务顺序执行。防止脏读、不可重复读、幻读问题

#### 传播特性

Propagation



##### required

​	默认传播特性，当前线程有事务加入当前事务，没有创建一个新事务。

##### supports

​	当前线程有事务加入当前事务，若没有事务不创建事务

##### mandatory（强制）

​	当前线程有事务加入当前事务，当前没有事务则抛出异常

##### requires_new

​	创建新事务，若当前线程存在事务则暂停当前事务

##### not_supported

​	执行无事务操作，如果当前线程存在事务则暂停当前事务

##### never

​	执行无事务操作，如果存在事务抛出异常

##### nested（嵌套）

​	

#### 

### 问题

#### 问题1

同一个类中调用申明了事务的方法，事务未生效，发生异常未回滚。



Spring事务管理通过AOP 代理去实现，也就是说Spring在初始化IOC容器时将事务管理加入bean（代理）中，所以如果事务产生作用首先要调用aop代理对象，而不是目标对象（this），如下



```java
@Component
public class ShopServiceTxTest {

    @Autowired
    private ApplicationContext context;

    @Autowired
    private ShopService shopService;

    @PostConstruct
    public void initAnnotationTx() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //调用目标对象方法，事务不会生效
                    updateByAnnotationTx();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }

    @Transactional(rollbackFor = Exception.class)
    public void updateByAnnotationTx() {
        String txName = TransactionSynchronizationManager.getCurrentTransactionName();
        log.info("---------- main transaction name is:{}.", txName);
        Shop shop = new Shop();
        shop.setId(2L);
        shop.setShopName("测试声明式事务");
        shopService.update(shop);
        throw new RuntimeException();
    }
}

public class ShopServiceImpl {
	@Transactional
    @Override
    public Long update(@RequestBody Shop shop) {
        String txName = TransactionSynchronizationManager.getCurrentTransactionName();
        log.info("---------- transaction name is:{}.", txName);
        System.out.println(shop.toString());
        this.entityDao.update(shop);
        return null;
    }    
}

```

代码中没有调用代理对象的updateByAnnotationTx方法，而是直接调用目标对象的方法，导致调用该方法时并没有事务管理，事务名txName为空null。

##### 原因

为什么会这样，spring在初始化ioc容器生成spring bean时，扫描到方法上有Transactional事务注解，会在bean对象的方法前后加上事务管理的代码，就是说只有使用spring通过代理生成的bean代理对象才会有事务，原始对象（目标对象）不会有事务。

##### 解决

获取ShopServiceTxTest的代理对象，调用代理对象的updateByAnnotationTx方法

```java

//1.上下文获取bean
context.getBean(ShopServiceTxTest.class).updateByAnnotationTx();
//2. 通过ThreadLocal 当前代理
((ShopServiceTxTest)AopContext.currentProxy()).updateByAnnotationTx();
```

​	

#### 问题2

spring bean是代理对象还是原生对象