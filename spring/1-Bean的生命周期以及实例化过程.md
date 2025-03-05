### Bean创建的整体过程

![bean创建的整体过程](.\bean创建的整体过程.png)

### Bean的生命周期

加载xml/注解的bean的描述信息

解析成Bean的定义信息BeanDifinition

调用BeanDifinition的增强器BeanFactoryPostPorocessor进行增强得到最终的BD对象

基于BeanDifinition对象【bean的定义信息】通过反射实例化bean

注入自定义属性值

注入容器对象属性值【实现了aware接口的bean】

调用BeanPostProcessor前置扩展方法postProcessBeforeInitialization

调用初始化方法，执行实现了InitializingBean接口，@PostConstruct注解的方法

调用BeanPostProcessor增强器的后置扩展方法 postProcessAfterInitialization

获取完整Bean对象

销毁流程-调用指定回调方法destory()



![bean生命周期](.\bean生命周期.png)

### Bean的实例化过程

申请内存空间



### Bean的初始化过程

注入自定义属性值

注入容器对象属性值

调用BeanPostProcessor前置扩展方法

调用初始化方法，执行实现了InitiazedBean接口，@PostConstruct注解的方法

调用BeanPostProcessor增强器的后置扩展方法

获取完整Bean对象

销毁流程-调用指定回调方法destory()

![bean初始化过程](.\bean初始化过程.png)