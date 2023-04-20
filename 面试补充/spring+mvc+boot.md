# Spring基础

## 重要的面向对象设计原则

开闭原则OCP

```
对扩展开放, 对修改关闭
```

依赖倒置原则

```
依赖抽象编程而不是依赖于具体(面向接口)
```

单一接口职责原则

```
接口/类的粒度尽可能小
```

合成复用原则

```
尽量使用组合和聚合而不是使用继承
```

里氏替换原则

```
子类在任何地方都可以替代父类
```

## 常用设计模式

单例模式

```
实现方法: 静态代码块, DCL+volatile, 枚举类, 
```

DCL代码

```java
private volatile static Singleton instance;
if (instance == null) {
	synchonized(Singleton.class){
		if (instance == null){
			instance = new Singleton();
		}
	}
}
```

工厂模式

```java
public class Factory{
	public Product getProduct(String name){
		if (name.equals("A")) return new A();
		else if(name.equals("B")) return new B();
	}
}
```

# IOC&DI

## BeanFactory

spring中的ioc容器负责bean的创建和管理, 当需要使用bean时之间拿来用就可以了

```
IOC容器指BeanFactory及其子类, 常用的是ApplicationContext, BeanFactory可以理解成一个HashMap<Name, Bean>, 而ApplicationContext在BeanFactory的基础上实现了许多额外的接口, 功能更多
ClassPathXmlApplicationContext是ApplicationContext的子类, 可以从类路径读取xml配置文件
```

Factorybean是什么?

```
通过配置文件向ioc容器中注册bean对于复杂对象太麻烦, 复杂对象可以实现FactoryBean<?>接口, 将接口实现类注册到ioc容器中, 获取bean时不会返回实现类, 而是返回接口的方法getObject()返回的bean对象
```

## Bean

bean生命周期

```
1. newInstance
2. setter注入属性
3. setBeanName()
4. setBeanFactory()
5. setApplicationContext()
6. 预初始化
7. 初始化
8. postProcessAfterInitialization
9. 可以被正常使用
10. PreDestroy
```

bean的作用域

```
1. 默认singleton
2. prototype 原型, 返回不同对象
```

bean依赖的注入方式

```
1. ByType: 查找符合类型的bean, 通过setter注入属性
2. ByName: 属性名和beanid一致就注入
3. Constructor注入: 在构造器上加@Autowired
```

## Resource 和 AutoWired

区别

```
@Autowired优先按照ByType方式注入
@Resource优先按照ByName方式注入
```

@Qualifier

```
@Qualifier: 显示声明需要注入的beanId
```

## 事务

Spring提供声明式事务管理, **底层实现原理是AOP**, 对方法执行前开启事务和执行后检查是回滚还是提交

```java
    @Transactional(isolation = Isolation.READ_COMMITTED)//设置事务隔离级别
    public void deposit(int money);
```

事务隔离级别和mysql中的一致

## 线程安全问题

ioc容器中如果单例bean被多个线程写入状态可能出现线程安全问题, 所以对可能出现线程安全问题的bean要设置为**原型模式/使用ThreadLocal作为bean**

## 循环依赖问题

如何解决循环依赖

```
原型bean的循环依赖无法解决, 单例的循环依赖可以通过在单例bean初始化后放到缓存中, 这样其他bean属性注入时可以使用没有初始化好的bean
```

三级缓存

```
一级缓存存放初始化好了的bean,
三级缓存存放刚实例化的bean
```

# AOP

## 基本名词

代码示例

```java
@Aspect
public class Advices {
//定义一个切入点
    @Pointcut("execution(void xyz.spider.helo.service.BookService.save())")
    private void pointA(){}

//切入内容
    @Before("pointA()")
    public void printTime(){
        System.out.println(System.currentTimeMillis());
    }
}
```

通知Advice

```
一般是一些公共的功能模块如日志, 用来扩展功能
通知类型: Before, After, Around, 异常通知, 返回通知
```

连接点joinpoint

```
可以使用通知的地方, 如方法执行前, 方法执行后, 方法抛出异常, 方法返回值
```

切入点Pointcut

```
一些特定连接点的集合
```

切面aspect

```
通知和切入点的集合, 可以理解成一层特定的功能切面
```

织入

```
将切面应用到目标对象并创建代理对象的过程
```

## 代理

JDK代理

```
JDk动态代理使用Proxy实现, 只能代理接口实现类
```

CGLIB

```
底层生成代理类的字节码并使用类加载器加载, 代理类是原类的子类
```

# SpringMVC

## 主要组件

![1673255572208](https://file+.vscode-resource.vscode-cdn.net/e%3A/%E6%B5%8F%E8%A7%88%E5%99%A8%E4%B8%8B%E8%BD%BD/permanent/sping/image/Spring/1673255572208.png)

```
1. DisPatcherServlet: 是一个Servlet, 拦截所有请求, 然后转发
2. HandlerMapping: 通过url映射到对应的Handler
3. HandlerAdapter: 处理请求并返回ModelAndView
4. ViewResolver: 解析ModelAndView并返回视图
```

## 常用注解

```
@RequestMapping
@RequestParam, @RequestBody, @PathVariable
@ResponseBody @RestfulController (序列化默认使用Jackson, 需要getter)

```

## 拦截器

拦截器可以在请求前和请求后进行处理, 相当于servlet中的filter, 一个拦截器类需要实现HandlerInterceptor

# SringBoot


## spring-boot-starter-parent

所有的springboot项目都继承自这个项目, **用于依赖项的版本管理**


## 自动配置原理

什么是自动配置?

```
别人jar包中写好的Java Config类会自动注册到IOC容器中
```

自动配置原理

```
@SpringBootApplication中@Import了一个AutoConfigurationImportSelector类, 这个类的作用是扫描类路径下的/META/spring.factories文件, 这个文件里包含了一些AutoConfiguration类的全类名

AutoConfiguration类会包含条件注解如@ConditionalOnClass, 包含条件注解的配置类在特定条件下才会注册为Bean
```

配置文件是如何生效的?

```
使用@ConfigurationProperties将配置文件中固定前缀的属性注入到Properties类中
@Value也可以将配置注入到配置类中

最后将Properties类注册为bean, 其他bean就可以直接使用配置了
```

springboot最重要的三个注解

```
ComponentScan
SpringBootApplication
EnableAutoConfiguration
```

# Mybatis

## #{}和${}区别

```
${} 使用字符串拼接的方式形成SQL, 可能会有SQL注入的问题
#{} 使用预编译SQL语句的方式防止了SQL注入
```

## 字段名和属性名不一致

使用ResultMap自定义映射规则可以设置property和column的对应关系

```xml
 <result property="studentName" column="student_name"/>
```

ResultMap还可以用来多列对一个属性(association),  一个集合属性对多行(collection)

```xml
    <association property="pub" javaType="com.wang.test.demo.entity.Publisher">
        <id property="id" column="p_id" jdbcType="VARCHAR"></id>
        <result property="name" column="name" jdbcType="VARCHAR"></result>
    </association>

<collection property="roleList" ofType="com.example.server.bean.Role" column="u_id" select="findRoleListByUserId"/>

```

## 模糊查询

如何进行模糊查询?

```
Like concat("%", #{key}, "%")
使用concat+#{}防止SQL注入
```
