# 1. IOC

## 1.1 IOC介绍

***IOC是什么?***

```
在一个对象需要使用被依赖对象时, 由IOC Service Provider(通常指IOC容器)主动提供被依赖对象, 而不是对象自己取获取被依赖对象
好处: 解耦了业务对象之间的绑定关系
```

![1688116593881](image/Spring&Mvc&Boot/1688116593881.png)

***IOC容器注入依赖的方式?***

```
1. 构造器注入
IOC容器负责管理被注入对象的构造过程, 会检查构造函数的参数列表并注入依赖
```

```
2. setter方法注入
IOC容器调用被注入对象的setter方法注入依赖
```

```
3. 接口注入(不提倡, 有侵入性代码)
被注入对象需要额外实现一个接口, 接口中包含注入依赖的方法, IOC容器调用注入依赖的方法实现依赖注入
```

***SpringIOC容器继承关系和对应的功能?***

* BeanFactory是spring中最基础的ioc容器(仅提供依赖管理和依赖注入服务)
* ApplicationContext继承自BeanFactory, 实现了额外的功能(如AOP, 事务管理)

![1688711961340](image/Spring&Mvc&Boot/1688711961340.png)

## 1.2 Bean的注册和BeanFactory的构造

***Bean注册过程中的重要类有哪些?***

* BeanDefinition: 包含一个Bean的所有依赖信息, 用于BeanDefinitionRegistry的构造
* BeanDefinitionRegistry: 包含多个Bean的依赖信息

![1688713649825](image/Spring&Mvc&Boot/1688713649825.png)

* AbstractBeanDefinitionReader: 从配置文件中读取出BeanDefinitionRegistry(如果使用xml配置文件则使用XmlBeanDefinitionReader)

![1688714338611](image/Spring&Mvc&Boot/1688714338611.png)

***BeanFactory的构造方式?***

1. 通过代码注册Bean, 构造BeanFactory

```java
    public BeanDefinitionRegistry getBeanDefinitionRegistryByCode() {
        BeanDefinition myBeanDefinition = new RootBeanDefinition(MyBean.class);
        BeanDefinitionRegistry beanDefinitionRegistry = new SimpleBeanDefinitionRegistry();
        beanDefinitionRegistry.registerBeanDefinition("myBeanId", myBeanDefinition);
        return beanDefinitionRegistry;
    }
```

2. 通过AbstractBeanDefinitionReader读取spring配置文件, 构造BeanFactory

```java

    public BeanDefinitionRegistry getBeanDefinitionRegistryByConfigurationFile() {
        AbstractBeanDefinitionReader reader = new XmlBeanDefinitionReader(new SimpleBeanDefinitionRegistry());
        reader.loadBeanDefinitions(new ClassPathResource("spring-config.xml"));
        return reader.getRegistry();
    }
```

3. 通过注解+包扫描

```java
@Component
public class MyBean {
    @Resource
    String name;
}
```

!!!包扫描程序会扫描指定包下的类然后构造BeanFactory, 如果检查到特定注解, 则自动确定依赖关系

```
// spring中通过xml配置文件开启包扫描
<context:component-scan base-package="com.Xxx.Xxx"></context:component-scan>

// springboot中通过注解@ComponentScan开启包扫描
```

***第三方Bean如何注册?***

* springboot中使用@Bean注解

```java
@Configuration
public class SpringBootConfig {
    @Bean
    public MyBean getBean() {
        MyBean myBean = new MyBean();
        // 通过代码的方式注入依赖到myBean...
        return myBean;
    }
}
```

* 配置文件将FactoryBean的实现类作为注册对象(`FactoryBean被注册到IOC容器时, IOC容器会将getObject()的返回值注册为Bean, 该bean的id为注册FactoryBean时配置文件中指明的id`)

```java

public class MyFactoryBean implements FactoryBean {

    @Override
    public MyBean getObject() throws Exception {
        MyBean myBean = new MyBean();
        // 使用代码注入依赖...
        return myBean;
    }

    @Override
    public Class<?> getObjectType() {
        return MyBean.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

## 1.3 Bean的使用

* bean为其他bean对象注入依赖

```java
@Component
public class MyBean {
    @Resource
    AnotherBean b;
}
```

* 如果在非bean对象中需要使用bean, 使用 `ApplicationContextAware` 实现类获取ioc容器, 再调用 `getBean()`方法

```
ApplicationContextAware的实现类在被注册到ioc容器中时, 会调用其setApplicationContext(当前ioc容器)
```

```java
@Component
public class MyAppContext implements ApplicationContextAware {
    public ApplicationContext applicationContext;
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    public ApplicationContext getApplicationContext() {
        return applicationContext;
    }
}
```

## 1.4 Bean的scope

***Bean的Scope有哪些, 具体作用?***

```
singleton(默认scope): 每次从容器中获取的bean是同一个对象
prototype: 每次从容器中获取的bean是新的对象

还有三个不常用且仅在WebApplicationContext中生效的scope:
request / session
```

## 1.5 Bean的生命周期

***bean的生命周期?***

```
1. IOC容器启动, 读取配置文件并生成Bean定义类
2. 通过反射实例化(instantiation)
3. 属性注入(Populate properties)
4. 调用各种Aware接口的方法
5. BeanPostProcessor的beforeInitialization方法
6. 如果实现了接口InitializingBean会调用afterPropertySet()方法
7. 调用初始化方法(init-method)
8. BeanPostProcessor的afterInitialization方法
-----

9. 调用destory-method
```

![1688965693587](image/Spring&Mvc&Boot/1688965693587.png)

### 1.5.1 BeanFactoryPostProcessor

在BeanFactory启动后会调用 `postProcessBeanFactory()`方法

```java
@Component
public class MyPostProcessor implements BeanFactoryPostProcessor, Ordered {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
    }
    @Override
    public int getOrder() {
        return 0;
    }
}
```

***s**pring提供的常见的BeanFactoryPostProcessor有哪些?***

```
PropertySourcesPlaceholderConfigurer: 替换注册信息中属性值为${占位符}的属性为.properties文件中的值
```

### 1.5.2 XxxAware

实现了XxxAware接口的bean会在属性注入结束后调用XxxAware接口定义的方法保存依赖

```java
@Component
public class ApplicationCtxManager implements ApplicationContextAware {
    public ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}

```

### 1.5.3 BeanPostProcssor

包含两个方法, 分别在bean初始化前后调用, 将 `BeanPostProcssor`注册到 `ApplicationContext`中即可生效

```java
@Component
public class MyBeanProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

### 1.5.4 init/destory方法

init()/destory()分别在bean初始化后, bean被销毁前调用

```java
@Component
public class MyBean {
    @PostConstruct
    public void init() {
        System.out.println("bean实例化完成");
    }

    @PreDestroy
    public void close() {
        System.out.println("bean即将被销毁");
    }
}
```

## 1.6 ApplicationContext的扩展功能

### 1.6.1 资源加载

**spring包下资源加载相关的重要类有哪些?**

![1688974417029](image/Spring&Mvc&Boot/1688974417029.png)

```java
// ResourceLoader可以加载一个资源
public interface ResourceLoader {
    String CLASSPATH_URL_PREFIX = "classpath:";

    Resource getResource(String location);

    @Nullable
    ClassLoader getClassLoader();
}
// ResourcePatternResolver可以根据pattern加载多个资源
public interface ResourcePatternResolver extends ResourceLoader {
    String CLASSPATH_ALL_URL_PREFIX = "classpath*:";

    Resource[] getResources(String locationPattern) throws IOException;
}
```

!!! `ApplicationContext`实现了 `ResourcePatternResolver`接口, 可以进行资源加载

### 1.6.2 国际化支持

***JAVA SE如何提供国际化支持?***

```java
// 通过两个重要的类Locale和ResourceBundle实现
//Locale
        String lang = "en"; // 英文地区
        String country = "US"; // 美国
        Locale us = new Locale(lang, country);
        Locale myLocal = Locale.CHINA; // new Locale("zh", "CN")

//ResourceBundle
        String baseName = "baseName";
        ResourceBundle resourceBundle = ResourceBundle.getBundle(baseName, Locale.CHINA);
        String key = "greeting";
        // 读取classpath:baseName_zh_CN.properties文件中的greeting属性
        String greetingMsg = resourceBundle.getString(key);
```

***Spring的如何提供国际化支持?***

使用 `MessageSource`代替 `ResourceBundle`的作用

![1688976947678](image/Spring&Mvc&Boot/1688976947678.png)

```java
@Component
public class MyClass implements MessageSourceAware {
    private MessageSource messageSource;
@Override
public void setMessageSource(MessageSource messageSource) { // 容器中需要先注册MessageSource
this.messageSource = messageSource;
}
    public void doSomething() {
        //获取key=greeting的值
String message = messageSource.getMessage("greeting", null, "Default Greeting", Locale.getDefault());
System.out.println(message);
}
}
```

!!! `ApplicationContext`实现了 ` MessageSource`接口

### 1.6.3 容器内事件处理

***JavaSE的事件监听机制?***

![1688978530441](image/Spring&Mvc&Boot/1688978530441.png)

***Spring容器的事件机制?***

`ApplicationEvent ApplicationListener ApplicationEventPublisher`分别继承了JavaSE中对应的类

ApplicationContext继承了 `ApplicationEventPublisher` , 当注册Bean时会记录所有的 `ApplicationListener`, 调用 `publishEvent()`时会发布给记录的Listner

下面是一个自定义事件监听器和发布器示例:

```java
@Component
public class MyEventListener implements ApplicationListener<MyEvent> {
    @Override
    public void onApplicationEvent(MyEvent event) {
        System.out.println("监听到我的自定义事件触发了, 时间触发时间:" +
                new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date(event.getTimestamp())));
    }
}
```

```java
@Component
public class MyEventPublisher implements ApplicationEventPublisherAware {
    public ApplicationEventPublisher evtPublisher;
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.evtPublisher = applicationEventPublisher;
    }

    public void publish(MyEvent event){
        this.evtPublisher.publishEvent(event);
    }
}
```

## 1.7 基于注解开发

在sprin配置文件中加入 `<context:component-scan>` 开启包扫描, 会扫描指定包下的所有类, **同时往容器中注册注解处理相关的BeanPostProcessor**

**@Autowired @Resource**

```
两者均可用于属性和方法上, 用于注入属性值(依靠反射机制)/方法参数/构造器参数
@Autowired默认ByType, 冲突时采用ByName
@Resource("beanId")默认byName, 没提供id使用ByType, 可以使用@Qualifier指明beanId
```

> !!! 当需要注入的字段类型为List/Map类型时, 会将容器内所有符合要求的类型都注入到List/Map(key为beanId)中

**@Component @Lazy**

```
@Component用于类上表明这是一个需要注册的bean, 可以使用带语义的@Service, @Controller...实现同样的效果

@Lazy指明一个bean是否懒加载(默认true, @Lazy(false)预加载)
```

**@Configuration @Bean**

```
@Configuration标注的类作为配置类(Configuration类会被注册到ioc容器中)(相当于一个xml配置文件)

@Bean标注的方法返回对象会被注册为Bean
```

**@Import**

```
如果一个被注册到ioc容器的中bean, bean.getClass().getAnnotations()包含@Import, 那么ioc容器会注册value中指明的class作为bean
```

@Import常常用于自定义的注解@EnableXxx, 用来手动注册模块(一个模块中包含多个需要注册的类)

```java
@Import({Blue.class, Red.class})
public @interface EnableColor {
}
```

# 2. AOP

## 2.1 AOP介绍

***AOP中的各种概念的含义?***

```
joinpoint: 可以切入的点
pointcut: 对一类可以切入的点的描述
advice: 需要切入的逻辑
aspect: 对pointcut和advice的模块化封装
wave: 切入的过程
waver: 执行切入过程的角色
target: 被切入的对象
```

![1689041613913](image/Spring&Mvc&Boot/1689041613913.png)

***Advice有哪些类型?***

```
Before Advice: 方法执行前的通知
After Returning Advice
After Throwing Advice
After Advice: 方法执行后的通知, 不论方法是否抛异常还是执行成功返回值
Around Advice
```

***Java中实现AOP常见的方法?***

```
使用JDK动态代理
使用CGLIB等动态字节码增强库实现动态代理
自定义类加载器: 类加载器在加载类时织入逻辑
直接生成java源码: 将要织入的逻辑代码直接生成到源码文件中
```

***Aop具体应用场景有哪些?***

```
* 日志/系统监控: 将方法的调用情况记录下来
* 缓存
* 事务管理
* 访问控制: 如果没有权限则拒绝访问
* 异常处理: 当发生uncheckedException时通知管理员
```

## 2.2 SpringAop原理

SpringAop采用动态代理的方式在代理对象中织入切面, 如果被代理对象实现类接口采用JDK动态代理, 否则采用CGLIB库实现动态代理

具体代理代码如下

![1689151389674](image/Spring&Mvc&Boot/1689151389674.png)

***JDK动态代理如何实现?***

```
关键方法Proxy.newProxyInstance(), 关键类InvocationHandler
!!!JDK通过动态代理生成的代理对象和被代理对象必须实现同样的接口
```

```java
public static void main(String[] args) {
    Subject subject = new Subject();
// JDK生成的代理对象会赋值给接口
ISubject iSubject = (ISubject) Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(),
Subject.class.getInterfaces(), new MyInvocationHandler(subject));
iSubject.hello();
}
static class MyInvocationHandler implements InvocationHandler {
    private Object target;
    public MyInvocationHandler(Object target) {
        this.target = target;
}
    @Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("前置通知");
Object ret = method.invoke(target, args);
System.out.println("后置通知");
        return ret;
}
}
```

***CGLIB动态代理如何实现?***

```
通过Enhancer生成一个类的子类对象(代理对象), 重写了非final方法用来织入切面 
```

![1689045466856](image/Spring&Mvc&Boot/1689045466856.png)

## 2.3 SpringAOP使用

1. 将一个POJO声明一个为一个Aspect
2. 开启 `@EnableAspectJAutoProxy`

```java
@Component
@Aspect
public class MyAspect {
    @Pointcut("execution(void spider.myspr.aop.Test.hello())")
    public void onMethodExecution() {}

    @Before("onMethodExecution()")
    public void beforeAdvice() {
        System.out.println("i will say hello");
    }

    @After("onMethodExecution()")
    public void afterAdvice() {
        System.out.println("i have said hello");
    }
}
```

***@EnableAspectJAutoProxy的作用?***

```
开启自动代理, 会往容器中注册一个AnnotationAwareAspectJAutoProxyCreator(是一个BeanPostProcessor)用于自动识别带@Aspect的bean并为Aspect类中描述的需要代理的对象生成代理对象, 也可以注册XxxAdvisor, 根据XxxAdvisor中描述的对象生成代理对象
```

***Aop失效的场景?***

```
* target对象中需要代理的方法是final方法, 会导致代理对象无法重写该方法, aop失效
* target对象中需要代理的方法A的方法体中调用了target对象中另一个需要代理的方法B, 在代理对象中调用A()会使得B()的通知失效
```

## 2.4 AOP和三级缓存机制

***什么是三级缓存?***

```
singletonObjects: 缓存初始化完成后的bean对象
earlySingletonObjects: 缓存提前进行AOP织入后的代理对象
singletonFactories: 缓存bean的ObjectFactory对象
```

***为什么是三级而不是二级缓存?***

如果只有循环依赖问题理论只需要二级缓存就可以解决, 引入 `earlySingtonObjects`的作用是缓存早期代理对象

![1689586509862](image/spring/1689586509862.png)

# 3. 事务管理

## 3.1 事务中重要概念

***事务的ACID特性指什么?***

```
* 原子性: 如果事务执行失败则回滚
* 一致性: 事务前后数据库的完整性没有被破坏
* 隔离性: 事务并发执行时可以保证不同执行顺序不影响最终的数据(通过事务的隔离级别实现)
* 持久性: 事务对数据库的修改是永久的
```

***事务的隔离级别有哪些?***

```
Isolation.READ_UNCOMMITTED : 事务期间其他事务写入的未提交的数据也有效-脏读问题

Isolation.READ_COMMITTED :  事务期间其他事务写入的提交后的数据有效-不可重复读问题

Isolation.REPEATABLE_READ: 事务期间不允许其他事务对同一个数据进行修改提交(增删可以提交)-幻读问题

Isolation.SERIALIZABLE : 事务期间不允许其他事务对同一个数据进行增删改提交-性能问题
```

***事务的传播行为指什么? 有哪些传播行为?***

在一个@Transactional方法A内部调用了另一个@Transactional方法B时, B方法的行为为传播行为

```
Propagation.REQUIRED: 当前没有事务新建事务, 当前有事务则加入
Propagation.SUPPORTS: 不新建事务, 当前有事务则加入

Propagation.REQUIRES_NEW: 新建一个事务, 如果之前有事务则挂起
Propagation.NESTED: 新建一个嵌套事务(嵌套事务失败时, 外层事务也失败)
Propagation.NOT_SUPPORTED: 不新建事务, 如果之前有事务则挂起


Propagation.MANDATORY: 方法执行前必须有一个事务, 否则抛异常
Propagation.NEVER: 方法执行前必须没有一个事务, 否则抛异常
```

## 3.2 Spring事务管理架构

***spring事务管理中重要的类以及类的作用?***

```
TransactionDefinition: 包含事务的隔离级别, 传播行为, 超时时间等事务定义信息
TransactionStatus: 事务状态
PlatformTransactionManager: 负责管理与线程绑定的连接上下文(jdbc中是Connection对象), 隔离了不同连接技术事务管理api的差异从而提供了统一的接口(不同的数据库连接技术对应不同的实现类, 如JDBC对应DataSourceTransactionManager)
```

![1689152886157](image/Spring&Mvc&Boot/1689152886157.png)

## 3.3 声明式事务和编程式事务

***声明式事务管理实现原理?***

```
包含被@Transactional标注的方法的对象, 会通过aop在方法执行前使用PlatformTransactionManager获取与当前线程绑定的数据库连接上下文(在jdbc中是Connection)并开启事务, 在方法内部所有dao与数据库连接都使用同一个连接上下文, 当方法结束时, PlatformTransactionManager根据是否抛出特定异常决定是否回滚事务, 最后关闭连接上下文并与当前线程解绑
```

***声明式事务什么情况下会失效?***

```
* 事务方法为final方法(aop失效)
* 事务方法被同一个对象里的其他方法调用时(aop失效)
* 事务方法为private方法时(spring规定的)
```

***编程式事务如何使用?适用情况?***

编程式事务适合多线程情况下

```java
PlatformTransactionManager transactionManager = ...;  // 根据具体数据库技术获取相应的txmanager
DefaultTransactionDefinition txDef = new DefaultTransactionDefinition();
txDef.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
// 获取事务状态
TransactionStatus txStatus = transactionManager.getTransaction(txDef);
        try {
            // 在此执行涉及数据库或其他资源的操作
  
            // 操作成功完成后，提交事务
            transactionManager.commit(txStatus);
        } catch (Exception ex) {
            // 发生异常时，回滚事务
            transactionManager.rollback(txStatus);
            throw ex;
        }
```

# 4. SpringMVC

## 4.1 原理图

***Springmvc基本组件有哪些, 各自的作用?***

![1673255572208](image/Spring/1673255572208.png)

```
DispatherServlet: 是一个Servlet, 拦截了所有请求, 并统一分发给特定的Handler(HandlerAdpater将Controller适配为Handler)

HandlerMapping: 负责请求到Handler的映射, 返回处理链(包含Handler和拦截器以及异常处理器)

Handler: 处理请求并返回ModelAndView

ViewResolver: 接受ModelAndView, 返回View, View的输出流写入响应体
```

***讲一下Springmvc处理链中的拦截器和异常解析器?***

```
拦截器:  在Handler处理请求前,后以及视图渲染完这三个时刻进行拦截

异常解析器: 处理器链中抛出了异常时异常解析器处理并返回默认的ModelAndView
```

![1689238009204](image/spring/1689238009204.png)

## 4.2 基于注解的开发

### 4.2.1 Controller

***Controller及其方法上常用的注解有哪些?***

```
>> @Controller @RestController(标注所有方法均为@ResponseBody方法)

>> @RequestMapping @GetMapping @PostMapping

>> @ResponseBody(标注方法返回值不用视图解析器解析, 直接走JackSon序列化器序列化为响应体)
```

> 易踩坑点: Jackson序列化对象时对象必须有getter方法

***没有被 `@ResponseBody`标注的方法的返回类型可以是什么?***

```
>> ModelAndView Model View

>> String (视图名称)
```

### 4.2.2 传递参数和依赖

***Controller的方法会自动注入依赖的参数类型有哪些?***

```
>> HttpServletRequest/HttpServletResponse/HttpSession/WebRequest: Servlet的请求响应类

>> InputStream/OutputStream/Reader/Writer: 请求或响应IO流

>> Map/ModelMap/ModelAndView: 要返回的ModelView对象

>> BindingResult: 注入参数被验证框架验证的结果
```

***query参数/form表单提交参数/multipart参数如何绑定?***

```
使用@RequestParam
```

***如何绑定路径参数?***

```
使用@PathVariable
```

***请求体如何绑定为方法参数?***

```
@RequestBody: 将请求体的JSON通过反序列化器映射为POJO
```

***如何添加请求中不存在但需要的绑定参数?***

```
实现HandlerMethodArgumentResolver子类, 并通过WebMvcConfigurer配置
```

## 4.3 MVC配置

> Tips: SpringMVC的配置类需要继承自WebMvcConfigurer)

***如何配置拦截器 `(HandlerInterceptor)`?***

```
重写配置类的addInterceptors()
```

***如何配置序列化器 `(HttpMessageConverters)`?***

```
注册类型为HttpMessageConverters的Bean
```

***如何配置参数解析器(`ArgumentResolvers`), 参数解析器的作用?***

```
重写配置类的addArgumentResolvers(), 参数解析器可以自定义Controller方法的参数绑定逻辑
```

***如何配置静态资源处理器(ResourceHandler)?***

```
重写配置类的addResourceHandlers()
```

> !!! 易踩坑点: 在通过前端路由实现的单页面应用中, 资源处理器应当:
>
> * 对所有请求路径不匹配/static/* 的请求, 返回index.html;

***如何处理跨域问题?***

```
浏览器根据同源策略, XHR请求中的域名如果和当前域名不同, 会拦截该跨域请求; 服务器默认也会拦截跨域请求(除非配置了允许的跨域)
```

解决方法:

```
>> SpringMVC服务器通过配置类的addCorsMappings()方法配置服务器允许的跨域

>> 浏览器可以通过JsonP等技术绕开跨域限制, 也可以直接使用允许跨域的浏览器
```

# 5. SpringBoot

## 5.1 springboot介绍

* springboot基于spring框架, 采用"约定大于配置"的方式简化了spring的配置(自动配置第三方依赖只需要一个 `spring-boot-starter-Xxx` jar包)
* `spring-boot-starter-web` 采用内嵌tomcat服务器
* 一个springboot应用的程序入口如下

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(MysprApplication.class, args);
}
}
```

## 5.2 @SpringBootApplication

***springboot自动配置原理?***

主启动类的注解 `@SpringBootApplication` 是一个组合注解, 其组成注解和作用包括

```
@SpringBootConfiguration: 启动类会被当做一个配置类注册
@ComponentScan: 进行包扫描, 注册bean(默认扫描启动类同包下的类)
@EnableAutoConfiguration: @Import导入EnableAutoConfigurationImportSelector
```

`EnableAutoConfigurationImportSelector` 这个类通过 `SpringFactoriesLoader`获取 `spring.factories` 文件中声明的配置类全类名列表, **并通过反射机制注册bean到容器中**

```java
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
```

> !!! 易踩坑点: jar包中的配置类无法通过包扫描注册

***jar包是什么, jar包约定的目录结构是什么?***

一个普通java程序被打包后形成的文件称为jar包, 可以通过 `java Xxx.jar`直接执行

jar包约定目录结构

* META-INF/
* com/demo/...或资源文件夹

```
* META-INF下包含jar包元信息, 主要是清单文件（MANIFEST.MF），其中包含了关于JAR包的版本、作者、依赖等信息
* ClassLoader.getResourceAsStream/loadClass默认起始路径为jar包的根路径
```

使用 `springboot-maven-plugin`打包后的jar包, `org/...`下是springboot主程序,  `BOOT-INF/lib`下包含依赖库的jar包使得打包后的jar包**可以直接独立运行**

> 易踩坑点: 如果打包后的jar包不直接包含依赖库的jar包, 需要运行的主机上 "jvm的classpath变量/环境变量CLASSPATH "   对应的路径下有依赖的jar包

![1689576469315](image/spring/1689576469315.png)

## 5.3 配置文件

***springboot如何导入配置文件中的属性值?***

使用 `@PropertySource() `导入配置文件的所有属性,  使用 `@Value/@ConfigurationProperties(prefix = "")`注入

```java
@PropertySource("classpath:my_application.yml")
@Configuration
public class AppConfig {
}

//----------------------------------------

    @Value("${myapp.property}")
    private String myProperty;

//----------------------------------------

@Component
@ConfigurationProperties(prefix = "myapp")
// 如果这个类没有被@Component标记, 需要其他被注册的类
// 标注@EnableConfigurationProperties({MyProperties.class})
public class MyProperties {
    private String property;
}
```

***多配置文件如何设置?***

主配置文件 `application.yml`中设置 `spring.profiles.active: Xxx` 启用从配置文件 `application-Xxx.yml`

# 6. SpringSecurity

***SpringSecurity的身份认证原理?***

```
* 基于Servlet容器的过滤器实现了一个过滤器链, 过滤器链获取Authentication对象放入SecurityContext中

* SecurityContext位于SecurityContextHolder中

* UserDetailsService通过访问数据库返回用户信息对象

* 登录时使用UserDetailsService.loadUserByUsername获取UserDetails, 生成JWT返回给前端, 并将JWT存入Redis

* 在过滤器链链中添加JWT认证过滤器, 获取请求头中的toke并验证token是否在redis中, token有效则生成Authentication对象放到SpringSecurityContext中
```