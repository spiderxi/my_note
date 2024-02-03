# 1. Java 简介

## 1. 术语

**_什么是 JDK & JRE ?_**

-   `JDK(java development kit) = JRE + Java开发工具(例如javac, javadoc)`
-   `JRE(java runtime enviroment) = 基于平台的JVM程序 + 动态链接文件 + 标准库class/jar文件 + 平台特有的class/jar 文件`

**_什么是 OpenJDK?_**

```
开源的的JDK, 相较于Oracle JDK而言(商用付费)
```

**_什么是 jar & war?_**

-   `jar(Java Archive) 是Java应用程序的压缩包 (包含class文件, resource文件, META-INF/文件下的元信息)`
-   `war 是 Java网络应用的压缩包 (额外包含/WEB-INF下的网络应用元信息和网络应用静态资源)`

**_什么是 classpath?_**

```
classpath指JVM中的ClassLoader加载类/资源的文件路径集合

Tip1: 通过ClassPath.getClassPath()查看
Tip2: classpath默认包含JAVA_HOME/lib/*.jar和Main.jar
```

## 2. 程序的启动和环境

**_讲一下 Java 源文件到程序运行的过程?_**

1. `使用javac编译源文件为class文件`
2. `使用jar打包class+资源文件+Manifest`
3. `使用java -jar Xxx.jar执行`

**_`java -jar Main.jar` 和 `java -cp <classpath> xyz.Main.class`的区别?_**

```
指定 classpath 的方式不同, -jar方式需要在 Manifest中指定 classpath
```

## 3. JDK

**_Java SE && EE && ME 的区别?_**

```
* SE(standard edition) 提供基础的开发库

* EE(enterprise edition) 是 SE(standard edition) 的超集

* ME(micro edition) 只提供移动端java开发库
```

**_JDK LTS 版本有哪些, 各自主要特性是什么?_**

| JDK | 主要特性                                 |
| --- | ---------------------------------------- |
| 8   | stream API, lambda 表达式, LocalDate API |
| 17  | zgc, seal 类, switch+lambda              |
| 21  | 虚拟线程                                 |

## 4. 语言概述

**_Java 和 CPP 的区别?_**

| Java             | cpp                          |
| ---------------- | ---------------------------- |
| **纯面向对象**   | 可以面向对象, 也可以面向过程 |
| **自动垃圾回收** | 使用 delete 手动回收垃圾     |
| 单继承+接口      | 多继承                       |
| **跨平台**       | 不跨平台                     |

# 2. Java 语言特性

## 1. 面向对象

**_Java 中的权限修饰符权限范围从大到小分别是什么?_**

`public` -> `default` -> `protected` -> `private`

**_方法重写 `(override)`和方法重载 `(overload)`的区别?_**

```
* 重写: 根据 方法调用者的实际类型 从 继承链往上 找到第一个的 同名同参 方法

* 重载: 根据 参数的外观类型 从方法列表中找到 同名 方法中最适合的一个方法
```

**_方法的参数个数不确定时, 如果传入参数?_**

```java
    void hello(String ...argus){
        System.out.println(argus[0]);
    }
```

**_Java 方法的参数传递中, 值传递和引用传递的区别?_**

```
* 基本类型传递另一份复制的变量

* 引用类型传递 被引用的源对象的地址
```

**_如何查看一个枚举类的所有枚举常量?_**

因为 `所有的枚举类都继承自Enum`, 所以使用 `Enum.values()`方法获取所有枚举常量

**_接口和抽象类的区别和相同点有哪些?_**

相同点

-   两者都不能被实例化

不同点

-   变量方面, 接口中所有变量为 `public static final`, 抽象类无限制
-   方法方面, 接口中所有方法为 `public abstract`, 抽象类无限制(可以包含 `abstract ` 方法)
-   构造器方面, 接口不能声明构造函数, **抽象类无限制**

## 2. 注解与反射

**_Java 中的元注解有哪些, 注解保留策略有哪些?_**

重要的元注解如下:

| 注解名            | 作用                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| @Override         | 标记重写方法                                                                                                                                           |
| @Deprecated       | 标记过时方法                                                                                                                                           |
| @SuppressWarnings | 忽略编译器警告                                                                                                                                         |
| **@Retention**    | **注解的保留策略**<br />**RetentionPolicy.SOURCE(只在源码中保留)<br />RetentionPolicy.CLASS(保留到编译期)<br />RetentionPolicy.RUNTIME(保留到运行时)** |
|                   |                                                                                                                                                        |
| **@Target**       | **注解用在什么地方**<br />ElementType.FIELD<br />ElementType.METHOD<br />ElementType.TYPE                                                              |
| **@Inherited**    | **注解是否会由父类传递给子类**                                                                                                                         |

**_编译期注解和运行期注解分别是如何生效的?_**

-   编译期注解通过 `AbstractProcessor` 生效, `AbstractProcessor` 中可以修改语法书
-   运行期注解通过反射 `Class#getAnnotations()` 生效

**_Java 中反射是什么, 反射为什么慢?_**

```
反射: 通过Class操作该类的对象

反射速度慢的原因: JIT无法对反射代码进行优化
```

**_有哪些方式能获取 Class 对象?_**

```
* Object#getClass()

* Object.class

* Class.forName()

* ClassLoader#loadClass()
```

## 3. 异常处理

**_讲一下 Java 异常处理类体系结构?_**

![1697613182265](image/java/1697613182265.png)

**_Exception 和 Error 的区别是什么?_**

```
Exception可以被程序捕获并处理, Error不能
```

**_什么是 Checked Exception?_**

```
必须被 try-catch处理的异常才能通过编译的异常, Exception的子类中除了 RuntimeException都属于 Checked Exception
```

**_什么是 `try-with-resource` 代码块?_**

```
Java7引入的语法糖, 在 try 代码块执行后自动关闭 Closeable接口实现类
```

**_finally 中的代码块一定会被执行吗?_**

```
不一定, 比如:
* catch代码块中使用System.exit()关闭JVM
```

**_如何绕过编译器的 `Checked Exception`检查?_**

```
* 使用Unsafe#throwException()
* 泛型方法中抛出泛型异常
```

## 5. 序列化

**_Java 序列化的方式有哪些?_**

```
* JDK序列化: 实现Serializable, 使用ObjectOutputStream

* 第三方库序列化: Gson, Jackson
```

**_transient 关键字的作用? serialVersionUID 的作用?_**

```
transient修饰的字段不会进行序列化

serialVersionUID: 使用JDK序列化时需要指定
```

**_序列化协议有哪些?_**

```
* XML
* JSON
* Protobuf
```

## 6. 泛型

**_泛型中泛型参数为 `? extends/super Xxx` 表示什么含义?_**

```
表明泛型参数必须是Xxx的子类或父类
```

**_什么是泛型擦除?_**

```
泛型参数在编译后会变成Object, 仅仅在获取泛型对象时添加强制类型转化指令
```

## 7. SPI

**_什么是 SPI?_**

`SPI(Service Provider Interface)` 是 Java 提供的扩展机制, 可以在运行时加载 `实现了特定接口的第三方服务实现类`

> SPI 典型应用: JDBC

**_怎样通过 SPI 加载第三方接口实现类?_**

```
1. 在jar包下/META-INF/services目录下添加映射文件

2. 使用ServiceLoader.load()方法
```

## 8. JDK 高版本新特性

**_Java 中的 Lambda 表达式是什么? (JDK8)_**

```
* Lambda表达式是一个语法糖, Lambda表达式可以赋值给函数式接口

* Lambda表达式: (arg0, arg1) -> {statement}
```

**_Stream 常用的中间操作和终止操作有哪些? (JDK8)_**

中间操作有 `filter(), map(), sorted()`

终止操作有 `forEach(), collect()`

## 9. IO 和文件

**_Java 中 BIO 中 IO 流分为哪四种?_**

```
IO流分四种:
 * InputStream OutPutStream
 * Reader Writer

这四种IO流的常用子类包括:
FileXxx
BufferedXxx
```

**_什么是 BIO, NIO, IO 多路复用, AIO?_**

```
* BIO: 线程阻塞直到操作系统IO完成

* NIO: 线程不阻塞, 立刻返回操作系统当前是否完成IO

* IO多路复用: 基于NIO, 操作系统同时对多个文件描述符进行监听, 线程阻塞一段时间并返回就绪的IO事件

* AIO: 线程不阻塞, IO完成后操作系统调用回调函数 or 发送信号通知进程
```

# 3. Java 容器

## 1. Collection

**_讲一下 Java 中的集合体系?_**

![1697619945103](image/java/1697619945103.png)

**_List 和 Set 的区别是什么?_**

```
List有序, 元素可以重复
Set无序, 元素会去重
```

## 2. List

**_ArrayList 和 LinkedList 的区别?_**

```
由于:
* ArrayList内部是一个Object[]

* LinkedList是一个Object双向链表

所以两者的随机读写和插入数据的时间复杂度会有区别
```

**_ArrayList 扩容机制是什么?_**

```
默认大小10, 每次扩容扩容到1.5倍空间并复制数据
```

**_ArrayList 多线程下修改会出现什么问题?_**

```
多线程下可能出现并发修改异常

Tips: 使用Vector代替ArrayList, Vector的方法为同步方法
```

## 3. Map

**_TreeMap 底层数据结构是什么?_**

```
红黑树
```

**_讲一下 HashMap 扩容机制?_**

```
* 初始大小为16
* 当表的负载因子超过0.75时, 扩容到两倍

Question: 为什么扩容到两倍?
Answer: hash值通过掩码的方式映射到桶索引
```

**_HashMap 从 JDK1.7 到 JDK1.8 改进了什么?_**

```
JDK8中HashMap哈希冲突过多/过少时(树化阈值8; 退化阈值6), 会在链表和红黑树之间转换
```

**_Hash 冲突的解决办法有哪些?_**

```
* 链表法

* 探测法: 在冲突位置向两侧按照一定算法(线性探测, 二次探测)进行探测, 找到一个空位置放置元素

* 公共溢出区: 将发生冲突的元素放到一个公共的区域(可以是链表, 也可以是另一个hash表(rehash法))

Tips: 探测法中, 当删除元素时, 需要对其他发生冲突的元素进行重定位
```

**_什么是一致性 hash 算法, 优点是什么?_**

优点

```
避免了桶的数量发生变化时, 需要进行大量的数据迁移
```

一致性 hash 算法原理

```
* hash函数的值域必须是一个环

* 桶节点和元素节点分别通过hash函数映射到环上一个点

* 通过映射点在环上最近的相邻映射点, 决定元素放到哪个桶中
```

**_HashTable 和 HashMap 的区别?_**

```
* HashTable线程安全

* HashTable处于兼容性考虑, 不可以使用null作为键
```

**_LinkedHashMap 是如何实现有序的?_**

```
LinkedHashMap内部的Entry继承自HashMap内部的Entry, 但额外包含前后指针, 通过指针维护链表
```

## 4. ConcurrentHashMap

**_ConcurrentHashmap 并发性能为什么好?_**

```
ConcurrentHashmap的锁粒度小:
* get()不加锁

* put()时, 最多只锁一个桶, 不会锁整个对象
	- 映射的桶为空时, 尝试cas
	- 映射的桶不为空时, 才获取对象锁
```

## 5. 阻塞队列

**_Java 中的常用的阻塞队列有哪些?_**

```
* ArrayBlockingQueue: 底层为数组

* LinkedBlockingQueue: 底层为链表, 可能会有OOM问题

* PriorityBlockingQueue: 按优先级出队列

* DelayQueue: 元素必须实现Delayed接口
```

## 6. 其他

**_Comparator 和 Comparable 的区别?_**

```
*  Comparator的比较方法的参数有两个

*  Comparable比较方法参数只有一个(用于比较自身和另一个Object的大小)
```

# 4. JDK API

## 1. Object

**_`Object#equals() `和 `== `的区别?_**

```
equals默认实现为==, ==比较两个Object是否为同一个引用
```

**_为什么重写 `equals()`必须要重写 `hashcode()`?_**

```
重写hashcode的是为了符合hash函数的定义(相同对象的hash值必须相等)
```

**_`Object#copy()`是深拷贝还是浅拷贝?_**

```
浅拷贝
```

## 2. String

**_如何改变 String 的编码?_**

1. 先调用 `String#getBytes方法`
2. 再调用 `new String(bytes, Charset)`

**_`new String("1" + "2" + new String("3"))`_** **_共创建了几个对象?_**

```
字符串常量池中两个对象: "12", "3"

堆区三个对象: 值为"123"的String, 值为"3"的String, StringBuilder
```

**_StringBuilder, StringBuffer, String 的区别?_**

```
* String内部是一个char[]

* StringBuilder是String的建造者类

* StringBuffer是线程安全的StringBuilder
```

**_`String#intern()`方法的作用?_**

```
将String对象放入字符串常量池, 并返回常量池中的引用

>>> 如果常量池中已有相等的对象, 直接该对象的引用
```

## 3. 包装类

**_Java 中的基本类型有哪些, 对应的包装类是什么, 各种类型的长度是多少?_**

| 基本类型 | 包装类  | 基本类型的长度(byte) | 字面值             |
| -------- | ------- | -------------------- | ------------------ |
| boolean  | Boolean | 依赖于 JVM 实现      |                    |
| byte     | Byte    | 1                    |                    |
| char     | Char    | 2                    |                    |
| short    | Short   | 2                    |                    |
| int      | Integer | 4                    |                    |
| float    | Float   | 4                    | `float f = 12.2F;` |
| double   | Double  | 8                    | `double d = 12.2D` |
| long     | Long    | 8                    | `long l = 12L;`    |

**_什么是自动装箱/自动拆箱?_**

```
* 自动装箱
	"Integer i = 1"  被编译为 "Integer i = Integer.valueOf(1)"
* 自动拆箱
	"int i = new Integer(1)" 被编译为 "int i = new Integer(1).intValue()"
```

**_为什么 `new Integer(11) == new Integer(11)`?_**

```
Integer中有一个内部类IntegerCache

当调用valueOf方法时如果-128<=值<=127会直接返回缓存的对象
```

## 4. 时间

**_为什么不推荐使用 Date 等旧时间 API?_**

```
旧API设计不合理, 太难用

Tips: 使用LocalDateTime代替Date
```

**_`LocalDateTime`和 `ZoneDateTime` 的区别?_**

```
ZoneDateTime额外保存时区ZoneId, 同一个时间戳时, 时间字符串会不同
```

## 5. 系统 API

**_Java 程序如何和 shell 交互(获取环境变量, 启动参数等操作)?_**

主要用到 `System`和 `Runtime`两个类

```
环境变量: System.getenv()

启动参数+宿主机环境变量: System.getProperties()

shell实体: System.console()

创建子进程执行命令: Runtime.getRuntime().exec()
```

**_`AccessController.doPrivileged()`方法的作用?_**

```java
无视JDK中SecurityManager的安全检查, 执行需要特定权限的代码
```

> JDK 的安全控制已经在 JDK17 后移除
