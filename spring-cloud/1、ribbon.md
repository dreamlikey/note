spring cloud对hystrix的集成在spring-cloud-netflix-core包中，spring boot启动加载spring.factories中Configuration对hystrix的核心类加载到spring中，包含有：

```
HystrixAutoConfiguration
HystrixSecurityAutoConfiguration
HystrixCircuitBreakerConfiguration
```

HystrixCircuitBreakerConfiguration声明HystrixCommandAspect 为spring bean 。

1、HystrixCommandAspect 是aop拦截所有标注了@HystrixCommand的方法，进行代理增强。



2、代理增强逻辑中，注解的属性封装成MetaHolder对象

通过这个对象初始化可执行的HystrixInvokable对象【其实就是将注解的信息解析封装成hystrix的AbstractCommand，表示当前方法执行hystrix逻辑的环境上下文【context】线程池、熔断器等】



3、hystrix上下文初始化完成后，执行逻辑CommandExecutor.execute()   -->  HystrixCommand#queue

--> toObservable()

toObservable() 响应式执行hystrix命令以及被代理方法



4、hystrix对执行逻辑的被观察者hystrixObservable添加了多个观察者，applyHystrixSemantics(final AbstractCommand<R> _cmd)创建运行主流程的被观察者



5、是否开启熔断，如果配置了circuitBreakerForceOpen表示熔断执行getFallback()逻辑；没有配置熔断走run()逻辑调用被代理方法逻辑