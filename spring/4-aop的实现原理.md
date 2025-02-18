spring aop的底层实现原理

### spring如何实现aop

Bean的初始化过程中，调用initmethod()后执行BeanPostProcessor增强器的后置处理【applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);】，处理的增强器中包含有aop的增强器AbstractAutoProxyCreator#postProcessAfterInitialization，获取bean的通知器【切面、通知】，对存在通知器的bean进行代理，代理实现方式分为两种JDK动态代理和CGLIB动态代理，如果bean实现了接口使用JDK动态代理，没有接口使用CGLIB。

### JDK动态代理实现

aop的jdk动态代理实现，spring解析出aop的切面和通知，在bean的每个method生成对应的代理方法，代理方法调用的invoke是JdkDynamicAopProxy.invoke()，这里需要动态解析代理方法的所有通知（advice）并添加方法拦截器MethodInterceptor，执行handler【JdkDynamicAopProxy】中的invoke()时会一次调用拦截器链中的通知方法，最后调用目标方法Method.invoke()



疑问：环绕通知如何实现？



### CGLIB动态代理实现