# 1. IOC

## 1. IOC简介

_使用 IOC 的好处?_

```
使用IOC容器管理对象之间的依赖关系, 解耦业务对象
```

_BeanFactory 和 FactoryBean 的关系?_

```
没有关系, BeanFactory是IOC容器的顶级接口, FactoryBean是注册Bean的一种方式
```

## 2. Bean

_讲一下bean的生命周期?_

```
1️⃣实例化: 通过反射调用无参构造方法new对象
2️⃣依赖注入: 从容器中获取依赖对象并注入字段值, 如果依赖对象未就绪则进行依赖对象的注册
3️⃣Aware接口回调: 如果bean实现了Aware接口, 则回调
4️⃣BeanPostProcessor前置回调执行
5️⃣init方法
6️⃣BeanPostProcessor后置回调执行
7️⃣destroy方法

🌙 BeanPostProcessor可以对注入依赖后的bean进行处理, 甚至可以替换bean, 是AOP/动态配置的实现原理
🌙 init方法中尝尝用于初始化资源(如网络连接), destroy方法中尝试释放资源 
🌙 Aware接口的作用是注入一些特殊依赖, 如ApplicationContext
🌙 依赖注入不是一个单独的阶段, 其依赖于InitDestroyAnnotationBeanPostProcessor(BeanPostProcessor子接口)实现, 实现该接口的Bean可扩展此生命周期
```

_Spring容器注入依赖对象的方式有哪些?_

```
🌟 构造器注入
🌟 setter方法注入
🌟 反射注入

🌙 当使用构造器注入时, 如果发生循环依赖会报错
```

_Bean常见的Scope有哪些?_

```
🌟 singleton(默认scope): 每次从容器中获取的bean是同一个对象
🌟 prototype: 每次从容器中获取的bean是新的对象
```

_懒加载和预加载Bean的区别?_
```
Bean为懒加载时, 会在getBean()时才会开始生命周期, 预加载Bean则会在容器启动时就开始
```

_BeanFactoryPostProcessor和BeanPostProcessor的区别?_

```
🌟 BeanPostProcessor: 可以在bean初始化前后进行拦截, 修改/替换bean实例, 
🌟 BeanFactoryPostProcessor: 在容器加载完Bean定义组装成BeanDefinitionRegistry后进行拦截, 修改/新增/删除Bean定义

🌙 两个PostProcessor都是Bean, 可以通过定义Bean进行扩展
🌙 bean的注册的时序为: Bean定义加载完成 => BeanFactoryPostProcessor注册就绪 => BeanPostProcessor注册就绪 =>  普通Bean注册就绪
🌙 BeanFactoryPostProcessor不能注入依赖, 因为依赖注入使用了BeanPostProcessor, 此时还没有就绪
🌙 先注册的BeanPostProcessor会对后续BeanPostProcessor的注册过程进行拦截, 所以进行依赖注入的BeanPostProcessor优先级很高
```

*如何实现从配置中心获取属性并添加到Spring的环境中?*
```
使用EnvironmentPostProcessor

🌙 EnvironmentPostProcessor不是Bean, Spring在启动时通过spring.factories SPI机制加载
🌙 Spring启动时序为: EnvironmentPostProcessor执行 => 加载application.properties并将属性值合并到Environment中 => 包扫描获取Bean定义 => BeanFactoryPostProcessor回调执行
```

## 3. 基于注解的开发

_@Autowired 和 @Resource 的区别?_

```
🌟 @Autowired: 优先使用ByType的方式注入
🌟 @Resource: 优先使用byID的方式注入, 默认字段名称为beanId

🌙 可以使用@Qualifier指定beanId
🌙 当需要注入的类型为Collection时, 会将容器内所有符合要求的类型都注入到容器中
```

_常用注解及其作用?_

```
🌟 @Component及其派生注解在包扫描时, 视作bean定义
🌟 @PostConstruct @PreDestory可以标识init/destory方法
🌟 @Bean可以将方法返回值注册为bean
🌟 @Import及其派生注解可以额外加载bean定义
🌟 @Value可以注入常量或使用占位符注入配置
🌟 @Scope可以标识bean的scope
🌟 @Lazy可以标识bean为懒加载

🌙 @Bean的实现原理为一个内置的BeanFactoryPostProcessor, 新增了bean定义并注册到BeanDefinitionRegistry中
🌙 @Value的实现原理为一个内置的BeanFactoryPostProcessor, 作用为替换bean定义中的占位符
```

*包扫描的实现原理?*
```
使用ASM(字节码解析)库, 扫描指定目录下的所有class文件, 分析类的元信息并封装成BeanDefinitionHolder
```

# 2. AOP

## 1. AOP简介

_AOP 中的各种术语的含义?_

```
🌟 Joinpoint: 可以切入的点 (例如某个方法)
🌟 Pointcut: 可以切入的点的集合 (例如一类方法)
🌟 Advice: 需要切入的逻辑
🌟 Aspect: Pointcut和Advice的组合
```

_Advice 有哪些类型?_

```
🌟 Before Advice: 方法执行前的逻辑
🌟 After Returning Advice: 方法正常返回的后置逻辑
🌟 After Throwing Advice: 方法抛出异常的后置逻辑
🌟 After Advice: 方法执行后的逻辑
🌟 Around Advice: 方法执行前后的逻辑
```

_Java 中实现 AOP 常见的方法?_

```
🌟 编译时直接修改字节码
🌟 运行时生成代理类(代理方式可以为JDK动态代理或CGLIB代理)
```

_Aop 具体应用场景有哪些?_

```
🌟 日志监控: 环绕通知, 上报方法入参和出参
🌟 事务管理: 例如声明式事务
🌟 权限控制: 前置通知, 如果没有权限则拒绝访问
🌟 统一异常处理: 异常后置通知, 捕获异常上报监控平台
```

## 2. SpringAOP 原理

_JDK动态代理如何实现?_
```
通过反射在运行时动态生成一个目标接口的子类

🌙 JDK动态代理有局限,直接代理特定接口的方法
🌙 SpringAOP优先使用JDK动态代理, 其次才是CGLIB代理
```

_CGLIB 动态代理如何实现?_
```
运行时生成字节码文件并使用类加载器加载到JVM中, 代理类是目标类的子类
```

_Spring AOP的实现原理?_
```
Spring识别到@EnableAspectJAutoProxy注解后, 会往容器中注册一个BeanPostProcessor(SmartInstantiationAwareBeanPostProcessor)用于自动识别带@Aspect的Bean, 并为Aspect涉及的Bean生成代理Bean

🌙 生成代理类的时机一般为BeanPostProcessor的后置回调, 但如果出现"循环依赖+AOP代理"的情况, 会在依赖注入期间就生成代理类
🌙 Bean注册优先级: BeanPostProcessor > Aspect > 需要AOP代理的普通Bean
```

_常见的AOP 失效的场景?_

```
🌟 AOP的目标方法为final方法时失效, 因为final方法无法代理
🌟 目标方法的调用方式为直接调用(如this.targetMethod()), 而不是通过bean调用(bean.targetMethod())
```

## 3. 三级缓存机制

_什么是三级缓存?_

```
🌟 一级缓存: 业务最终使用的Bean, 已完成AOP代理和依赖注入
🌟 二级缓存: 提前暴露的Bean, 已完成AOP代理, 不一定完成依赖注入
🌟 三级缓存: ObjectFactory, ObjectFactory内部包含实例化后的Bean对象, 但在getBean调用时会

🌙 二级缓存可以解决循环依赖的问题(A -> B -> A), 引入三级缓存是为了解决"AOP代理+循环依赖"场景的问题
🌙 依赖注入时, 先检索一级缓存, 没有时检索二级缓存, 还没有检索三级缓存获取ObjectFactory, ObjectFactory方法内部会进行AOP代理并返回代理对象
🌙 解决AOP代理的核心思想为延迟调用ObjectFactory获取代理后的对象
🌙 ObjectFactory检测是否需要AOP代理依赖于SmartInstantiationAwareBeanPostProcessor(BeanPostProcessor子接口), 可扩展
```

## 4. 声明式事务
_编程式事务和声明式事务的区别?_
```
编程式事务: 手动调用事务管理API
声明式事务: 通过AOP实现
```

_Spring声明式事务实现原理?_
```
基于AOP和JDBC原生事务管理API实现

1️⃣方法执行前: 使用JDBC DataSource获取Connection实例, 通过ThreadLocal绑定线程和Connection实例
2️⃣方法执行中: 通过Spring 提供的DataSourceUtils工具类获取Connection实例
3️⃣方法执行后: 通过ThreadLocal获取Connection实例, 提交事务或回滚, 然后释放Connection实例

🌙 @Transactional方法内部手动获取的Connection, Spring声明式事务无效
🌙 @Transactional方法内如果开启子线程进行JDBC操作, Spring声明式事务无效
```

# 3. SpringMVC

## 1. 原理

_SpringMVC抽象层组件有哪些?_

```
🌟 DispatcherServlet: SpringMVC入口, 基于Tomcat
🌟 HandlerMapping
🌟 HandlerInterceptor: 在Hanlde处理前后拦截
🌟 HandlerMethodArgumentResolver: Handler参数自动绑定
🌟 Handler: 实际请求处理类, 返回ModelAndView
🌟 HandlerMethodReturnValueHandler: Handler返回值处理
🌟 ViewResolver: 从ModelAndView中解析出View

🌙 SpringMVC基于tomcat
🌙 核心架构思想为"责任链模式"
```

## 2. 注解开发
_HandlerAdaptor类适配源类常用的注解?_
```
🌟 @RestController
🌟 @ResponseBody
🌟 @RequestMapping/@PostMapping/@GetMapping

🌙 ResponseBody的原理是基于HandlerMethodReturnValueHandler
```

_SpringMVC默认使用哪个库进行序列化?_

```
默认使用Jackson进行序列化

🌙 Jackson序列化时需要getter方法
🌙 通过WebMvcConfigurer添加HttpMessageConverter可以扩展序列化器
```

_Handler方法的哪些参数类型会被SpringMVC自动绑定?_

```
🌟 Servlet相关类: HttpServletRequest/HttpServletResponse/InputStream/OutputStream
🌟 ModelAndView: 要返回的ModelView对象
🌟 BindingResult: 参数验证的结果
🌟 @RequestParam: query参数, form参数,  multipart参数
🌟 @PathVariable: path参数
🌟 @RequestBody: 请求体反序列化参数

🌙 参数的自动绑定依赖参数解析器Bean(HandlerMethodArgumentResolver)实现, 可以通过注册自定义Bean扩展
```
## 3. MVC 配置

_如何配置SpringMVC?_

```
配置类的接口为WebMvcConfigurer

🌙 WebMvcConfigurer可以配置拦截器, 异常解析器, 参数解析器, 跨域规则
```

_前端为SPA(Single Page Application)时, SpringMVC 如何配置资源路径映射?_
```
将大部分路径(/**)都映射到index.html
```

# 4. SpringBoot

## 1. 介绍

_为什么要使用 SpringBoot?_

```
使用第三方库的xxx-starter.jar包时可以自动注册bean, 无需手动注册
```

## 2. 实现原理

_SpringBoot 自动配置原理?_
```
使用了类似于Java SPI的机制:
SpringBooth启动时读取三方jar包中的spring.facotries文件, 加载中文件列出的自动配置类, 配置类被@Conditional派生注解时只有在符合条件时才被注册为Bean

🌙 检查是否符合条件时会使用@Conditional指明的Condition子类
🌙 @Conditional注解原理为BeanFacotoryPostProcessor
```



