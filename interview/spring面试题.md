## 1、spring的IOC/DI是指什么

​      spring IOC是一种思想，是指控制反转，就是说将创建对象的控制权进行转移，关注的是对象创建的问题。
​      还有一个DI的概念，关注的是对象之间依赖关系的问题。
​      spring IOC/DI将对象的创建和依赖交给IOC容器管理。在传统的java程序设计中，我们要使用对象都是通过new进行创建，通过程序主动去依赖对象，比如设置某个属性，通过setxxx方法进行设置。而spring IOC容器解放了这个传统创建对象和设置依赖关系的动作。需要什么样的对象，我们按照相关规则配置好，需要依赖什么，也配置即可。那么启动容器的时候，容器会主动为我们创建对象和注入依赖关系，达到松耦合的效果。



## 2、AOP是什么

AOP是面向切面编程，是一种开发理念，是oop编程的补充。可以用做事务管理，日志管理等功能。



## 3、AOP的框架有哪些，简单介绍下

​     aop框架基于代理模式，分为静态代理和动态代理.有基于动态代理的spring aop框架和基于静态代理的apsecj框架。

​     spring aop致力于解决企业级开发中最普遍的方法织入。

- spring AOP


  动态代理有两种类型，分别为jdk的动态代理和cglib的动态代理。

  - jdk动态代理

    需要实现一个接口，因为生成的代理对象本身已经时继承了Proxy类的，且java不支持多继承的，所以通过接口的方式来实现。

  - cglib动态代理

    基于类进行代理，生成的类时继承于目标类的，所以被代理的类不能时final类型的，方法也不能时private或者时final的。

    

  性能上由于基于动态代理实现，所以需要生成代理对象，性能相对于aspectj要差点。

  

- aspectj

  静态织入，通过修改代码来实现，有以下几个织入的时机：

  - 1、编译期间织入：将切面类和目标类放在一起进行编译
  - 2、编译后织入：目标类已经编译成class文件，此时使用命令织入aop代码
  - 3、类加载后织入：在jvm加载类的时候，做字节码替换

  apectj在实际运行之前就完成了织入，所以它生成的类时没有额外开销的。

  

## 4、JDK动态代理和CGLIB动态代理的区别

​       都是基于动态代理的方式进行织入，实例化一个代理对象，来完成对目标的代理。

​       jdk动态代理代理的类是一个接口，如果不是接口，spring会使用cglib的方式进行代理。cglib因为通过继承的方式进行代理，所以代理的目标类不能是一个final类型的，对应的方法不能是final或者private类型。cglib的比较复杂，相对来说性能差一点。



## 5、spring如何选择使用哪种代理模式

- 1、如果目标对象实现了接口，则默认采用JDK动态代理；
- 2、如果目标对象没有实现接口，则使用Cglib代理；
- 3、如果目标对象实现了接口，但强制使用了Cglib，则使用Cglib进行代理

对应选择的源码：

```java
 public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
        if (!config.isOptimize() && !config.isProxyTargetClass() && !this.hasNoUserSuppliedProxyInterfaces(config)) {
            return new JdkDynamicAopProxy(config);
        } else {
            Class<?> targetClass = config.getTargetClass();
            if (targetClass == null) {
                throw new AopConfigException("TargetSource cannot determine target class: Either an interface or a target is required for proxy creation.");
            } else {
                return (AopProxy)(!targetClass.isInterface() && !Proxy.isProxyClass(targetClass) ? new ObjenesisCglibAopProxy(config) : new JdkDynamicAopProxy(config));
            }
        }
    }
```





## 6、简单介绍下Spring的事务传播

事务传播行为是事务对方法是否起作用，多个方法调用的时候，事务是沿用旧事务，还是新开事务的问题。

对应的枚举源码：

```java
public enum Propagation {
   
    REQUIRED(0), 
    SUPPORTS(1), 
    MANDATORY(2), 
    REQUIRES_NEW(3),
    NOT_SUPPORTED(4),
    NEVER(5),  
    NESTED(6);  
    private final int value;
 
    private Propagation(int value) {
        this.value = value;
    }
 
    public int value() {
        return this.value;
    }
}
```

- ① REQUIRED(0)：（默认值） 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中置。
- ② SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行
- ③ MANDATORY：强制性使用事务，调用具有该事务行为的方法，如果没有事务，会抛出异常
- ④ REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。
- ⑤ NOT_SUPPORTED：表示当前方法不支持事务，有事务会挂起，在方法里不起作用
- ⑥ NEVER：表示调用该事务行为的方法，有事务的话，会抛出异常
- ⑦ NESTED：嵌套事务，就是被调用的那个方法里的事务跟当前方法是相关的，如果当前方法抛出异常，那么被调用的方法也会回滚事务，如果被调用的方法出现异常，当前方法会回到调用方法之前的位置。



## 7、Spring支持的几种bean的作用域

- singleton: 单例
- prototype： 多实例
- request： 一个请求一个实例
- session： 一个会话一个实例
- global session：  全局session作用域



## 9、spring 的事务隔离级别

- DEFAULT: 使用数据库默认的事务隔离级别
- 读未提交：脏读，不可重复读，幻读
- 读已提交：不可重复读，幻读
- 可重复读： 避免脏读和不可重复读，幻读还是有可能发生
- 串行化： 性能不太好。



## 10、解释一下Spring AOP里面的概念

- 切面（aspect）:   可以理解平时我们用@Aspect注解修饰的类就是一个切面
- 目标对象（targate）: 需要被增强的对象
- 连接点（JoinPoint）:  程序被拦截的点。spring中因为只能对方法织入，这里指的是被拦截的方法
- 切入点（PointCut）:  @PointCut注解定义的就是切入点，切入点就是提供一种规则来匹配连接点。
- 通知（Advice）: 通知指拦截到连接点之后要执行的代码。
- 织入 (Weaving):  这个其实是目标对象生产代理对象的过程，也就是将需要添加的功能和实际的功能，一起放入到代理对象中，就是个织人的过程。



## 11、spring 通知类型有哪些

- 前置通知 @Before:   不管方式是否出现异常，都会执行前置通知。

- 后置通知 @After  发生异常也是会执行。

- 异常通知 @AfterThrowing  后置异常通知，当方法抛出异常的时候，会执行。

- 后置返回通知：@AfterReturning, 当方法抛出异常的时候，不执行。

- 环绕通知 @Around   上述通知的集合，拥有上述所有的功能。

  

## 12、spring bean的生命周期

BeanDefintion ->  MergeBeanDefinition ->  实例化前 -> 实例化 -> 实例化后 -> 属性填充 ->  init方法前 -> init -> init方法后 -> 返回bean



## 13、spring框架中用到什么设计模式

工厂模式IOC容器就是一个大工厂；动态代理，比如AOP和事务的功能；观察者模式：事件功能。



## 14、spring创建bean的方式

- xml方式
- 注解方式，@Service、@Controller、@Component、@Repository等
- @Configuration的方式
- FactoryBean的方式
- @Import方式
- @ImportSelector方式



## 15、Spring框架中有哪些不同类型的事件，你怎么自定义事件

- 上下文更新事件（ContextRefreshedEvent）：初始化或者更新的时候
- 上下文开始事件（ContextStartedEvent）： 调用start的时候
- 上下文停止事件（ContextStoppedEvent）：调用stop的时候
- 上下文关闭事件（ContextClosedEvent）： 容器关闭的时候
- 请求处理事件（RequestHandledEvent）:   web运用中，一个请求结束后触发这个事件

自定义的事件很简单，可以直接继承ApplicationEvent，编写需要监听的逻辑，然后实现ApplicationListner，可以泛型绑定前面的Event，可以在重写方法中，通过类型判断来监听实际的事件。

```java
AnnotationConfigApplicationContext ap
                = new AnnotationConfigApplicationContext(Config.class);
        ap.start();
        ap.stop();
        ap.close();        
```

触发事件的结果：

```java
org.springframework.context.event.ContextRefreshedEvent
org.springframework.context.event.ContextStartedEvent
org.springframework.context.event.ContextStoppedEvent
org.springframework.context.event.ContextClosedEvent
```





## 16、springmvc的流程

​    MVC：分别代表module、view和controller

- 用户发送请求到dispatcheservlet
- dispatcherSevlete收到请求后调用HandlerMapping处理映射器
- 处理映射器找到具体的处理器，生成处理器对象和处理拦截器返回给dispatchServlet
- dispatchServlet调用HandlerAdapter处理适配器
- 经过适配调用具体的后端控制器（就是controller了）
- controller执行业务代码，返回ModelAndView
- HandlerAdapter将ModelAndView返回给dispatchServlet
- dipatchServlet将ModelAndView传给试图解析器Viewsolver
- Viewsolver解析后返回具体的View
- dipatcherservlet得到view之后进行渲染视图
- dispatchservlet将响应返回给前端



## 17、BeanFactory 和 ApplicationContext有什么区别

ApplicationContext实现了BeanFactory接口，拥有BeanFactory所有的功能，同时还提供了资源访问，事件发布，国际化等功能。一般都推荐使用ApplicationContext，因为功能比较齐全。



## 18、BeanFactory和FactoryBean有什么区别

BeanFactory是一个bean工厂，用来管理bean的，就是IOC容器。FactoryBean是工厂bean,提供一种生成bean的方式

