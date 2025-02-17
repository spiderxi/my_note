# 1. JVM Memory Model

## 1. 运行时数据区

**_JVM运行时数据区分为哪些?_**

```
🌟 Heap
🌟 PC(program counter register)
🌟 JVM Stacks: 由Stack Frame组成, 一个Stack Frame保存了一次方法调用的上下文
🌟 Native Method Stacks: 本地方法(Native Method)的栈
🌟 MetaSpace: 存储类的元信息/运行时常量池

🌙 其中PC/虚拟机栈/本地方法栈/Heap上的TLAB为线程私有
🌙 MetaSpace使用Directory Memory
```
**_什么是 TLAB?_**
```
Heap区线程私有的空间, 避免多线程下分配空间要使用CAS降低效率
```

***什么是Direct Memory?***
```
JVM进程向操作系统申请的内存, 相较于JVM的Heap区而言Direct Memory不会被GC(JDK4前Direct Memory又称为Native Memory)

🌙 通过Unsafe.allocateMemory或ByteBuffer.allocateDirect可以申请Direct Memory
🌙 当DirectByteBuffer对象被GC时, 对应的Direct Memory也会被归还给操作系统
🌙 可以通过JVM参数设置Direct Memory最大大小
```


**_栈帧的具体结构?_**
```
🌟局部变量表
🌟返回地址
🌟操作数栈
🌟动态链接指针
```

***如何设置各内存区域大小?***

```
-Xss: 栈大小
-Xmx -Xms: 堆区大小
-XX: MaxMetaspaceSize: 元空间大小
-XX: MaxDirectMemorySize: 直接内存大小
```

***OOM如何排查?***

```
1️⃣使用JDK工具jmap导出JVM内存快照
2️⃣使用可视化工具查看快照, 根据GC引用链查看内存泄漏原因
```

## 2. 对象的存储和创建

**_对象在堆区的存储结构是怎样的?_**
```
1️⃣对象头(GC信息+锁信息+哈希码+类型指针)
2️⃣对象体
3️⃣对齐空间

❓为什么要对齐
👄便于检索
```

**_new一个对象会发生什么?_**

```
1️⃣ 加载相关类和父类
2️⃣ 在Heap区的TLAB上分配空间
3️⃣ 初始化对象体为0
4️⃣ 设置对象头
5️⃣ 调用<init>函数
```

`<init>`**_函数的执行步骤?_**

```
1️⃣ 调用父类的<init>函数
2️⃣ 按照变量声明顺序初始化变量的值, 并执行初始化代码块
3️⃣ 执行构造函数
```


# 2. 类

## 1. 类加载过程

**_class 文件的结构?_**
```
OXCAFEBABE + Java版本号 + 静态常量池 + 类的字段定义 + 方法列表
```

**_什么情况会触发一个类的加载?_**
```
🌟主动引用类时(访问构造函数/static方法/static变量)
🌟加载子类时
```

***_类加载的过程?_**
```

```

## 2. 类加载器

**_类加载器的层级结构?_**

```
从上到下为:
1️⃣ 启动类加载器(BoothStrap ClassLoader): 加载<JAVA_HOME>\lib下的类
2️⃣ 扩展类加载器(Ext ClassLoader): 加载<JAVA_HOME>\lib\ext下的类
3️⃣ 应用程序类加载器(Application ClassLoader): 加载ClassPath下的类
4️⃣ 自定义类加载器: ClassLoader的实现类
```

**_什么是双亲委派机制?_**

```
当下一级的类加载器loadClass()时, 会先调用上一级的类加载器的loadClass()方法尝试加载

🌙上一级的类加载器通过组合的方式组合到下一级类加载器中
🌙可以通过重写loadClass()破坏双亲委派机制
🌙双亲委派机制是出于安全考虑(避免三方类覆盖了JDK中的类)
```


**_可以自己写一个 java.util.String 并加载用来覆盖 JDK 中的 java.util.String 吗?_**

```
不可以, 双亲委派机制导致不会加载重复的类, 即使跳过双亲委派机制内部defineClass()也不允许加载包名为java.*的类

🌙JVM中, 一个类的ID通过类加载器和全限定名共同确定
```

## 3. 方法的调用和执行


**_什么静态分派(Static Dispatch) 和 动态分派?_**
```
🌟静态分派(重载): Java编译器根据方法参数的静态类型(又称外观类型) 找到要调用的具体方法的符号引用

🌟动态分派(重写):` JVM运行时通过方法的调用者的实际类型, 来确定要调用的方法
```
**_invokevirtual指令的执行过程是怎样的?_**
```
`invokevirtual + 方法的符号引用`时, 会获取操作数栈顶调用者的实际类型, 然后从子类到父类逐级查找方法, 直到抛出异常
```


## 4. Java 编译器

**_什么是 JIT?_**
```
JVM执行系统分为 Java 解释器和 JIT(即时编译器):

🌟   Java 解释器负责将字节码执行翻译成机器指令并执行
🌟   JIT对频繁执行的方法会直接缓存成机器指令, 加快执行速度
```


**_什么是 Java 泛型擦除?_**
```
Java泛型的实现机制, 编译后的字节码指令中会把泛型类型参数变成Object, 只有在对泛型变量进行读写时进行强制类型转换和类型检查

🌙可以通过反射绕过泛型检查
```

# 3. GC

## 1. GC分区

**_Heap区中是如何分代的?_**

```
🌟一个Eden区
🌟两个Survivor区
🌟一个Old区

🌙 80%的对象朝生夕灭, 所以对Heap区按对象年龄进行划分
```

**_为什么有两个Survivor区?_**
```
Eden区GC算法为标记复制
```
**_给对象分配堆空间的流程?_**
```
1️⃣ 优先Eden区分配
2️⃣ Eden区内存不足时触发YGC, 清理Eden区和一个S区, 存活对象放到另一个S区(对象在S区YGC次数>15时也会自动晋升到Old区)
3️⃣ S区内存不足时部分对象晋升到Old区
4️⃣ Old区内存不足时触发FGC

🌙 大对象会直接放到Old区
```


## 2. GC算法

**_垃圾标记算法有哪些?_**

```
🌟 引用计数法(reference count): 会造成循环计数问题
🌟 可达性分析(reachable anaysis): 从GC ROOT顺着引用链看看是否可以到达对象
```

**_GC ROOT 对象有哪些?_**

```
局部变量, static变量, 活跃的线程
```

**_垃圾回收算法有哪些?_**

```
🌟 标记清除: 不用移动对象, 但会产生内存碎片, 适合Old区
🌟 标记复制: 需要浪费额外的区域存放存活的对象并花时间复制存活对象, 一般适合Eden区等垃圾较多的区域
🌟 标记压缩: 需要移动大量对象, 但节约内存, 适合Old区
```

## 3. Garbage Collector

**_各版本JDK的Garbage Collector分别是什么?_**
|JDK版本|GC Collector|
|-|-|
|JDK5|支持CMS|
|JDK8|支持G1|
|JDK17|支持ZGC|


**_CMS(Concurrent mark sweep)的特点?_**
```
🌟Old区采用标记清除算法
🌟二次标记垃圾和清除垃圾时不会STW(GC线程和用户线程并发)
```

**_G1(Garbage First)的特点?_**
```
🌟Old区采用标记复制算法
🌟将堆区分为更小的Region, 每个Region再分为Eden/Old/S/H区,GC时可以控制要GC的Region从而实现控制STW的时间
🌟更多的Region需要消耗内存空间管理(空间换时间策略)
```


**_ZGC的特点?_**

```
🌟 STW的时间和堆大小无关, O(1)时间复杂度, STW时间小于10ms
🌟 空间换时间策略
🌟 实现原理: 指针染色+读屏障
```

## 4. GC API

**_System.gc()会立刻触发GC吗?_**

```
System.gc()调用后不会立刻执行gc, JVM自行判断GC时机(只是建议系统进行GC)
```

**_Java 中的四种引用类型是什么?_**

```
🌟 强引用: 永远不会被回收
🌟 软引用: 内存不足时被回收, 用于缓存
🌟 弱引用: 只要发生GC就会被回收, 用于缓存
🌟 虚引用: 用于对象被回收时获取通知
```
