分析的spring的版本为：5.1.6.RELEASE版本



通过AnnotationConfigUtils抽象类讲下面的处理器注入到register中

```
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
```



注册BeanDefinitionHolder： BeanDefinition的持有者

```
BeanDefinitionHolder
```



`Bean的放入方法`

```java
protected void addSingleton(String beanName, Object singletonObject) {
		synchronized (this.singletonObjects) {
			this.singletonObjects.put(beanName, singletonObject);
			this.singletonFactories.remove(beanName);
			this.earlySingletonObjects.remove(beanName);
			this.registeredSingletons.add(beanName);
		}
	}
```



加入bean是：

```java
调用：prepareBeanFactory(ConfigurableListableBeanFactory beanFactory)时
environment ->   StandardEnvironment
systemProperties ->  Map<String, Object>
systemEnvironment -> Map<String, Object>

调用： invokeBeanFactoryPostProcessors
1、org.springframework.context.annotation.internalConfigurationAnnotationProcessor
->   ConfigurationClassPostProcessor

2、org.springframework.context.annotation.internalAutowiredAnnotationProcessor
->   AutowiredAnnotationBeanPostProcessor

3、org.springframework.context.annotation.internalCommonAnnotationProcessor
->   CommonAnnotationBeanPostProcessor

4、org.springframework.context.event.internalEventListenerProcessor
->   EventListenerMethodProcessor

5、org.springframework.context.event.internalEventListenerFactory
->   DefaultEventListenerFactory 


执行了invokeBeanDefinitionRegistryPostProcessors()方法后就会执行1的后置处理方法
会解析Configuration注解的类
对于@Bean的方法，会先生成一个BeanDefinitionMap,key为方法名，值为一个内部类
ConfigurationClassBeanDefinitionReader$ConfigurationClassBeanDefinition对象

ConfigurationClassBeanDefinition extends RootBeanDefinition implements AnnotatedBeanDefinition
```



在initMessageSource()，会加入一个bean:

```
messageSource -->  DelegatingMessageSource
```

在initApplicationEventMulticaster()，会加入一个bean

```
applicationEventMulticaster -> SimpleApplicationEventMulticaster
```





`Bean的生命周期`

- BeanDefinition
- BeanDefinition合并
- 加载类
- 推断构造方法
- 实例化
- BeanDefinition的后置处理
- 属性填充
- 执行Aware
- 初始化前
- 初始化
- 初始化后





注解的形式：

```java
	private final AnnotatedBeanDefinitionReader reader;

	private final ClassPathBeanDefinitionScanner scanner;
```





