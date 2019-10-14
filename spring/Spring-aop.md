Spring面向切面编程



#### AOP概念

##### Aspect 切面

切面 = 切入点+通知，**在什么时机，什么地方，做什么增强**

##### Join point 连接点

程序执行的入口，如方法的执行或异常的处理，在Spring AOP中连接点往往指的是方法的执行

##### Advice通知

在切面的连接点上采取的动作，通知类型有环绕通知（around）、前通知（before）、后通知（after），许多AOP框架包括Spring将Advice定义为lterceptor拦截器，并在连接点join point周围简历拦截器链

##### Pointcut 切入点

匹配连接点的关键词，通知与切入点表达式相关联，在与切入点Pointcut匹配的任何连接点join point上运行（拦截匹配pointcut表达是的方法）。由切入点表达式匹配连接点的概念是aop的核心，Spring默认使用aspectj切入点表达式。

##### Introduction

通过其他方式声明方法或字段

##### Target object

被一个或多个切面通知的对象，也称为“被通知对象”，由于Spring aop是通过代理实现的，所以这个对象一直是代理对象

##### AOP proxy 代理

由AOP框架创建的对象，用来实现切面所要完成的工作（执行通知方法等等），在Spring中通过jdk动态代理、CGLIB来实现代理

##### Weaving 织入

**把切面加入到对象，并创建代理对象的过程**



#### AOP proxy 代理

Spring aop默认为aop代理使用jdk动态代理实现，它允许代理任何一个或一组接口

Spring aop还可以使用CGLIB进行代理，在spring中使用CGLID来代理没有实现接口的类，程序员可以对接口强制使用CGLID代理

