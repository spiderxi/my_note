# 1. Jvm内存模型

## 1. 运行时数据区

***Jvm运行时数据区分为哪些?哪些是线程私有的?***

```
* 堆
* 程序计数器
* 虚拟机栈
* 本地方法栈
* 元空间(JDK8前被称为堆区中的方法区, JDK8后被称为Metaspace并使用本地内存实现)

其中程序计数器, 虚拟机栈私有
```

![1678537054081](https://file+.vscode-resource.vscode-cdn.net/e%3A/permanent/java/image/jvm/1678537054081.png)

***栈帧的具体结构是什么?***

虚拟机栈是由栈帧组成的栈, 每个被调用的方法对应一个栈帧, 栈帧由 `局部变量表, 返回地址, 操作数栈, 动态链接(一个指针, 指向该方法在运行时常量池中的直接引用)` 组成,

其中局部变量表由 `slot`组成, 一般类型变量占用一个 `slot`, `double/long`占用两个, **局部变量表的大小在编译后可以完全确定**

**其中动态链接用于支持执行方法时的分派**

![1678792483331](https://file+.vscode-resource.vscode-cdn.net/e%3A/permanent/java/image/jvm/1678792483331.png)

***运行时数据区会出现哪些异常, 如何排查和避免?***

```
* 虚拟机栈超过最大调用深度: StackOverFlow
* 虚拟机栈超过最大大小: OOM
* 堆区超过最大大小: OOM
* 元空间大小超过最大大小: OOM
* 直接内存超过最大大小: OOM
```

设置各内存区域大小可以避免异常

```
-Xss: 栈大小
-Xmx -Xms: 堆区大小
-XX: MaxMetaspaceSize: 元空间大小
-XX: MaxDirectMemorySize: 直接内存大小
```

出现了OOM排查步骤:

```
1. 使用工具将内存dump下来
2. 使用可视化工具查看快照, 根据GC引用链查看是否发生了内存泄漏(部分对象不会被使用, 但没有从GC引用链上移除)
3. 如果只是内存溢出(内存不足), 修改jvm内存配置参数/提高硬件配置
```

***本地内存和直接内存的区别?***

* 本地内存是操作系统分配给Jvm进程的内存空间但不被Jvm管理, 被操作系统管理的空间
* 直接内存是本地内存的一部分, 用于Java NIO, 避免IO流的数据在本地内存和Jvm内存直接频繁复制

***什么是TLAB?***

堆区线程私有的空间, 用于避免多线程下堆区分配对象空间要使用**CAS**

***内存泄漏/频繁发生FGC如何排查?***

```
1. 查看OOM信息, 确定是哪个内存区域报错
2. 使用JVM参数设置OOM时自动生成内存快照的dump文件 或者使用 JDK工具jmap
3. 通过dump文件和可视化工具查看是哪些对象内存多大, 是发生了内存泄漏还是内存溢出
4. 修改相关代码 /  增加硬件配置
```


## 2. 普通对象的存储和创建

***对象在堆区的存储结构是怎样的?***

`对象头(GC信息+锁信息+哈希码+类型的指针) + 对象体 + 对齐空间`

***new对象之后的过程是怎样的?***

```
1. 先加载new对象的类及其父类
2. 在堆区的TLAB上分配空间
3. 初始化对象体为0
4. 设置对象头
5. 调用<init>函数
```

`<init>`***函数的内容?***

```
1. 调用父类的<init>函数
2. 按照变量声明顺序初始化变量的值, 过程中如果遇到{初始化代码块}会执行初始化代码块
3. 执行构造函数
```

## 3. 字符串常量池

***什么是String.intern()?***

如果字符串常量池中存在值和当前String对象值相同的String, 则返回该常量引用, 否则将当前对象放入常量池中并返回该常量引用

***new String("a" + "b")创建了多少对象, new String("a") + new String("b")呢, String s = "abc"呢?***

```
new String("a" + "b"): 
创建了两个对象, 分别字符串常量池中的"ab"(编译器优化), 在堆区中值为"ab"的String对象

String ab = new String("a") + new String("b"): 
6个对象, 分别是StringBuilder, 字符串常量池中的"a" & "b", 值为"a"/"b"/"ab"的在堆区的String对象

String s = "abc":
创建了一个对象, s指向字符串常量池中的"abc"
```

# 2. 类

## 1. 类加载过程

***class文件的结构?***

大致的结构为: `OXCAFEBABE + Java版本号 + 静态常量池 + 类的字段,方法列表`

***类加载过程和加载时机?***

**对一个类的主动引用**会触发类加载, 主动引用包括:

```
* 访问static变量/方法
* 初始化类的对象
* 加载子类
```

加载过程分为 `加载 -> 验证 -> 准备 -> 解析 -> 初始化`

```
加载: 类加载器的loadClass方法, 通过类全限定名将二进制流加载到元空间并在堆区生成Class对象

验证: 验证类文件是否合法

准备: 给static变量分配空间并设置为0, 给static final变量赋值

解析: 将常量池中的部分符号引用解析为直接引用

初始化: 执行器<cinit>方法
```

***cinit函数的执行过程, 多个类加载器同时加载类会重复调用cinit函数吗?***

cinit函数只会被执行一次, 执行过程如下

```
1. 给static变量赋值
2. 执行static代码块
```

## 2. 类加载器

***类加载器有哪几类?***

```
* 启动类加载器: 加载<JAVA_HOME>\lib下的类
* 扩展类加载器: 加载<JAVA_HOME>\lib\ext下的类
* 应用程序类加载器: 加载ClassPath下的类
* 自定义类加载器: ClassLoader的实现类
```

***什么是双亲委派机制, 如何破坏?***

```
当下一级的类加载器 loadClass()时, 会调用上一级的类加载器的 loadClass()方法尝试加载, 加载失败再自己加载, 上一级的类加载器通过组合的方式组合到下一级类加载器中
```

```
破坏双亲委派机制可以 重写loadClass()
```

> ! Jvm中, 一个类的ID通过类加载器和全限定名共同确定

***可以自己写一个java.util.String并加载用来覆盖JDK中的java.util.String吗?***

```
不可以, 使用三类自带类加载器会因为双亲委派机制失败, 即使使用重写了loadClass的类加载器也会因为调用了defineClass()导致失败, defineClass不允许加载包名为java.*的类
```

## 3. 方法的调用和执行

***什么是方法调用中的解析(Resolution)?***

JVM编译期间**将可以唯一确定的方法(private/static方法)的符号引用替换为直接引用**

***什么静态分派(Static Dispatch) 和 动态分派?***

`静态分派(重载原理):` Java编译器根据被调用的成员方法的参数的静态类型(又称外观类型) 找到要调用的具体方法的符号引用

`动态分派(重写原理):` Jvm运行时通过被调用的成员方法的句柄的实际类型, 来确定 `invokevirtual`

要执行的具体方法

***invokevirtual指令是什么, 执行过程是怎样的?***

invokevirtual是jvm字节码指令, 执行 `invokevirtual + 方法的符号引用` 时, 会获取操作数栈顶调用者的实际类型, 然后从子类到父类逐级查找**可访问**的方法, 直到抛出异常

***子类和父类有同名变量且都为public, 子类方法中访问的同名变量是谁的?***

子类对象中子类变量会覆盖父类变量, 子类方法中访问同名变量是子类的变量

## 4. Java编译器

***什么是native方法?什么是JIT?***

虚拟机执行系统分为Java解释器和JIT(即时编译器)

* Java解释器负责将字节码执行翻译成机器指令并执行
* JIT对频繁执行的代码块/方法, 直接翻译成机器指令, 加快解释器执行速度

***Lombok生效原理?***

lombok使用了编译器插件, 编译器在编译时对语法树进行了修改, 具体来说就是继承了 `AbstractProcessor` 调用了相关API

```Java
@SupportedAnnotationTypes("AddX")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class AddXProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        for (TypeElement annotation : annotations) {
            for (Element element : roundEnv.getElementsAnnotatedWith(annotation)) {
                if (element.getKind() == ElementKind.FIELD && element.getModifiers().contains(Modifier.STATIC)) {
                    String variableName = element.getSimpleName().toString();
                    String newVariableName = variableName + "X";
                    processingEnv.getMessager().printMessage(Diagnostic.Kind.NOTE, "Adding X suffix to variable: " + variableName);
                    modifyVariableName(element, newVariableName);
                }
            }
        }
        return true;
    }
}
```

***什么是Java泛型擦除?***

Java泛型的实现机制是泛型擦除, 即编译后的字节码指令中会**把所有类型参数变成Object**, 只有在对泛型变量**进行读写时进行强制类型转换**

这个泛型机制导致了泛型不能使用基本类型, 泛型丢失等问题

***自动装箱/拆箱/for-each循环语法糖实现原理是什么?***

自动装箱在编译后变成调用 `Integer.valueOf()`

自动拆箱在编译后变成调用 `integer.intValue()`

for-each循环变成了调用 `collection.iterator()`

***i++和++i在字节码指令层面的区别?***

i++

```
load i
incr i, 1
```

++i

```
incre i, 1
load i
```

# 3. 垃圾回收

## 1. 堆的分区

***堆中是如何针对垃圾回收进行分区的, 为什么要进行分代?***

```
堆区分为 Eden区 两个Survivor区 Old区
分代是为了提高GC效率, 大部分对象朝生夕灭, 所以分配在Eden区, Eden区GC更加频繁
```

![1678800922245](https://file+.vscode-resource.vscode-cdn.net/e%3A/permanent/java/image/jvm/1678800922245.png)

***为什么需要两个S区?***

Eden区垃圾回收算法使用**标记复制算法**

***一个对象被分配堆空间的流程(GC角度)?***

![1678871311411](https://file+.vscode-resource.vscode-cdn.net/e%3A/permanent/java/image/jvm/1678871311411.png)

***Eden区频繁发生YGC/FGC如何解决?***

```
1. 查看是否发生了内存泄漏/垃圾代码中频繁地创建对象
2. 如果代码没有问题, 使用Jvm参数-XX:NewRatio增大Eden区大小
```

## 2. 垃圾回收相关算法

***垃圾标记算法有哪些?***

```
* 引用计数法: 会造成循环计数问题
* 可达性分析: 从GC ROOT顺着引用链看看是否可以到达对象
```

***GC ROOT对象有哪些?***

```
局部变量, static变量 
```

***垃圾回收算法有哪些?***

```
* 标记清除: 不用移动对象, 但会产生内存碎片, 适合Old区
* 标记复制: 需要浪费额外的区域存放存活的对象并花时间复制存活对象, 一般适合Eden区等垃圾较多的区域
* 标记压缩: 需要移动大量对象, 但节约内存, 适合Old区
```

## 3. 垃圾回收器

***JDK8中默认的垃圾回收器搭配是什么?***

Eden Old区采用并行垃圾收集器

![1692349233837](image/Java虚拟机/1692349233837.png)

***CMS垃圾回收器回收过程和优缺点?***

CMS全称并发标记清除, 过程为

```
1. STW, 初始标记
2. 并发标记垃圾
3. STW, 重新标记
4. 并发清除
```

![1692348820568](image/Java虚拟机/1692348820568.png)

优点是 `STW时间短`,  缺点是 `会产生内存碎片`

***G1垃圾回收器的原理和优缺点?***

G1全称Garbage First

```
原理: 
* 将堆区分为更小的Region, 每个Region再分为Eden/Old/S/H区
* 根据限制的停顿时间, 动态选择回收价值更大的部分分区回收
* 回收过程采用标记复制算法
```

优点: `可以控制停顿的时间` 缺点: `G1使用内存较大`

***什么是ZGC?***

```
JDK11中引入的垃圾回收器, 停顿时间可以达到O(1)级别1ms以内
```

## 5. 垃圾回收相关API

***System.gc()什么时候执行, finalize()方法会有什么问题?***

```
System.gc()调用后不会立刻执行gc, Jvm自行判断gc执行的时间

finalize()不正当的使用会导致对象复活
```

***Java中的四种引用类型是什么?***

```
* 强引用: 永远不会被回收
* 软引用: 内存不足时被回收, 用于缓存
* 弱引用: 只要发生GC, 都会被回收
* 虚引用: 用于对象被回收时应用程序获取通知
```
