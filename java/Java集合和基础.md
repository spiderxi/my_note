# 1. Java基础

## 1.1 方法

方法重载: 多个同名方法共存

```java
    //方法名相同, 参数列表不同, 返回值可以不同, 修饰符可以不同
    public int getSum(int a, int b){
        return a+b;
    }
    public double getSum(double a){
        return a+1.3;
    }
```

方法重写(override): 覆盖父类的方法

```java
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("hello, Override!");
            }
        };
```

方法的参数传递

* 基本数据类型传递值, 引用类型传递引用(可以理解为单层指针)
* 参数不能使用默认值, 默认值通过重载方法实现
* 可以使用不定项参数

```java
    public void printParameterLength(int... args){
        System.out.println(args.length);//args是数组
    }
```

常用方法

```java
        System.out.println("hello,world!");
        /**
         * == 判断引用的是否是同一个对象
         * 一般重写equals方法比较对象逻辑上是否相同
         */
        this.equals(null);
        /**
         * clone可以进行对象的浅复制
         * native方法比new一个对象再复制快得多
         */
        Object obj = this.clone();
         /**
         * 对象被当作垃圾要回收前时调用的一个函数, 小心使用, 防止对象复活
         */
        this.finalize();
	//instanceof 判断对象是否属于某个类/子类
        new String() instanceof Object
```

## 1.2 数组

```java
//******************************************初始化
//创建大小为10的数组(所有元素默认值为int的默认值0)
int[] arr = new int[10];
//创建指向空的数组
int[] arr = null;
//列表初始化
int[] arr = {1, 1, 2};

//******************************************遍历
//利用下标和循环遍历数组
for(int i = 0; i < arr.length; i++){
    System.out.println(arr[i]);
}
//foreach循环遍历数组
for (int num : arr) {
    System.out.println(num);
}

//******************************************操作
//java.util.Arrays类提供static方法对数组操作
//初始化(全部初始化为1)
Arrays.fill(arr, 1);
//打印数组的所有元素的值
System.out.println(Arrays.toString(arr)); 
//升序排序(数组的元素要实现Comparable接口)
Arrays.sort(arr);
//复制数组
int[] newArr = null;
Arrays.copyOf(newArr, arr.length);
//比较两个数组的所有元素是否相等
Arrays.equals(arr, newArr);
//二分查找
Arrays.binarySearch(arr, 2);//返回元素值为2的索引

//二维数组
//2x3的数组声明
int[][] gird = new int[2][3];
//初始化
int[][] gird = {{3, 4}, {1, 2}};
```

## 1.3 枚举类Enum

所有枚举类是java.lang.Enum的子类

声明一个枚举类

```java
public enum Color{
	//与普通类声明不同的一点: 首先给出所有枚举常量
	BLUE("蓝色"),
	GREEN("绿色");
	//和普通类一样有方法, 属性, 构造器
	private String des;
	public Color(String des){
		this.des = des;
	}
	public void say(){
		System.out.println(des);
	}
}
```

如果有需要查看所有枚举常量可以使用从Enum类中继承过来的方法:

`Color[] allColors = Color.values();`

## 1.4 注解

Java中的注解分为7个元注解(Java语言自带的注解)和程序员自定义的注解.

* 元注解

元注解各自的作用如下

| 注解名               | 作用                         |
| -------------------- | ---------------------------- |
| @Override            | 告诉编译器这个方法是重写的   |
| @Deprecated          | 告诉编译器这个内容是过时的   |
| @SuppressWarnings    | 告诉编译器忽略Warning        |
| **@Retention** | 声明自定义注解的有效范围     |
| @Documented          | 声明自定义注解是否加入文档   |
| **@Target**    | 声明自定义注解的使用范围     |
| **@Inherited** | 声明自定义注解的是否有继承性 |

* 自定义的注解

自定义的注解通常是利用反射机制来搭框架,

声明语法为

```java
@Documented//要用元注解声明自定义注解的属性
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnotation{//与声明接口相同, 只是关键字interface前加了@
	String[] value() default "unknown";
}
```

* 反射与运行期生效注解

假设通过反射获得了某个Method对象, 他是被MyAnnotation注解过的.

那么有以下与注解有关的操作, 可以让我们获得注解标注的内容并进行相应操作

```java
method.isAnnotationPresent(MyAnnotation.class)//看Method是否有特定注解
MyAnnotation myAnnotation = method.getAnnotation(MyAnnotation.class);
//获取方法的注解
String[] values = myAnnotation.value();//获取注解的属性值
```

* APT与编译期生效注解

Annotation Processing Tool，简称APT，是Java提供给开发者的用于在编译期对注解进行处理的一系列API，这类API的使用被广泛的用于各种框架，如lombok. **APT可以在编译期间处理语法树**

> 通过继承AbstractProcessor实现自定义编译期生效注释处理器

## 1.5 异常处理

* java异常接口为Throwable, 实现类为Error(程序无法处理的问题)和Exception(程序可以显式处理的问题)
* 异常处理代码

```java
try {
    逻辑程序块
} catch(ExceptionType1 e) {
    处理代码块1
} catch (ExceptionType2 e) {
    处理代码块2
    throw(e);    // 再抛出这个"异常"
} finally {
    释放资源代码块
}
```

* 异常抛出

```java
    public void test() throws Exception {//方法内部不处理异常时, 向上抛出异常
        throw(new Exception("错误信息"));//抛出异常
    }
```

* 打印异常栈

```java
new Exception().printStackTrace();
```

## 1. 6 Math&String

```java
//格式化字符串
String.format("dp[%d][%d] = %d", j, K, minMaxPart)
//string byte[]互相转化
byte[] b = "hello, world!".getBytes();
System.out.println(new String(b));
//String重写了equals(), 可以比较两个字符串逻辑是否相等
"??".equals("??");
       	//1. 常量
        //自然对数e
        double e = Math.E;
        //圆周率pi
        double pi = Math.PI;

        //2. 各种数学函数
        Math.min(1, 2);
        Math.max(3, 4);
        Math.abs(-1);
        Math.ceil(2.3);//向上取整
        Math.floor(2.3);//向下取整
        //支持三角函数sin, cos, tan, 以及对应的反三角函数
        Math.sin(Math.PI);//弧度制
        Math.acos(1);//值域[0, pi]
        //弧度和角度相互转换
        System.out.println( Math.toDegrees(Math.PI));
        System.out.println(Math.toRadians(180)); 
```

## 1.7 Lambda函数

Java中Lambda函数是依附于函数式接口(只有一个抽象方法的接口叫做函数式接口)存在的,  进一步来说**Lambda函数是用来给(函数式)接口实例化的.**

```java
        Runnable r = () -> {System.out.println("你好");};
        //上面的式子右边是一个Lambda函数
        //这个式子的含义是将r中run()方法实现为右边的Lambda方法
        //Lambda方法只能实现只有一个函数没实现的接口
  
        //Lambda函数的语法:
        //1. ->的左边是参数, ->的右边是函数体
        //2. 参数不用指明类型
        //3. 函数体可以是一个{}括起来的代码块, 也可以是一句代码, 可以有返回值.

        (a, b) -> {return a+b;}//合法的Lambda函数
	a -> a+1 //只有一个参数可以简写, 返回值为a+1
```

java自带了一些常用函数式接口如下

```java
       Consumer c = new Consumer<T>() {
            @Override
            public void accept(T t) {
                // TODO Auto-generated method stub
  
            }
        };
        Supplier s = new Supplier<T>() {
            @Override
            public T get() {
                // TODO Auto-generated method stub
                return null;
            }
        };
        Function f = new Function<T,R>() {
            @Override
            public R apply(T t) {
                // TODO Auto-generated method stub
                return null;
            }
        };
        Predicate p = new Predicate<T>() {
            @Override
            public boolean test(T t) {
                // TODO Auto-generated method stub
                return false;
            }
        };
```

## 1.8 Stream API

使用Stream API分为获取stream, 操作stream, 收集stream

获取stream

```java
        //1. Collection及其子类使用.stream()获取
        List<Integer> list = new ArrayList<Integer>();
        list.stream();

        //2. 数组使用Arrays.stream()获取
        int[] arr = {1, 5, 10};
        Arrays.stream(arr);

        //3. java.util.stream.Stream中的of()方法可以产生一个stream
        Stream<Integer> s = Stream.of(1, 2, 3);
```

操作stream

```java
        String[] arr = {"hello", "world", "hi", "interesting"};
        //###Stream的中间操作###

        //##过滤
        //1 .filter()的参数是一个Predicate, 返回值为false的元素会被去除掉
        Arrays.stream(arr).filter(a -> a%2 != 0);//去除掉偶数
        //2. skip(i)去除掉流的前i个元素
        Arrays.stream(arr).skip(1);
        //3. limit(n)流的最大长度为n, 多余n的元素被去除掉
        Arrays.stream(arr).limit(2);
        //4. 通过hash值去除重复元素
        Arrays.stream(arr).distinct();

        //##映射
        Arrays.stream(arr).map(a -> a.length()); //map()的参数是一个Function

        //##排序
        //sorted()不加参数是自然排序
        Arrays.stream(arr).sorted();
        //添加Comparator可以使用Comparator的compare方法进行比较
        //Comparator也是一个函数式接口
        Arrays.stream(arr).sorted((a, b) -> a.length() - b.length()).forEach(System.out::println);
```

stream的收集操作

```java
        //1. forEach迭代
        Arrays.stream(arr).forEach(System.out::println);
        //2. 收集到一个容器
        Stream<String> stream = Stream.of("aaa", "bbb", "ccc", "bbb");
        List<String> list = stream.collect(Collectors.toList());
```

**stream没有调用终止操作时中间操作也不会执行!**

## 1.9 SPI

SPI(Service Provider Interface), 是Java提供的一种服务发现机制, 可以通过接口查找相关实现类并动态加载实现类达到解耦+框架组件替换, 例如JDBC中的java.sql.Driver接口不同厂商会有不同实现

如何实现服务发现机制?

```
jar包下的META-INF/services/中有配置文件, 配置文件中包含要实现的接口和实现类之间的映射关系

通过ServiceLoader发现并加载接口的实现类
```

ServiceLoader的代码

```java
        // 加载实现了JDBC Driver接口的类
        ServiceLoader<Driver> serviceLoader = ServiceLoader.load(Driver.class);
        serviceLoader.forEach((impl) -> {
            System.out.println(impl.jdbcCompliant());
        });
```

# 2. Java集合

Java集合的体系

![1679860238832](image/Java集合和基础/1679860238832.png)

## 2.1 Collection

迭代器遍历

```java
        Collection<String> collection = new ArrayList<String>(){{add("jack"); add("jerry");}};
        // Collection的迭代器遍历
        Iterator iterator = collection.iterator();
        while (iterator.hasNext()){
            System.out.println(iterator.next());
        }
        //增强for遍历Collection本质是使用迭代器
        for (String s: collection){
            System.out.println(s);
        }
```

## 2.2 ArrayList&Vector&LinkedList

### 2.2.1 ArrayList

* **ArrayList底层使用Object数组**
  ```java
  transient Object[] elementData; // non-private to simplify nested class access
  ```
* ArrayLsit线程不安全, Vector线程安全

默认数组长度==0,  第一次add()时会设置数组长度为10

```java
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
```

**数组空间不足时扩容到1.5倍**, 扩容后超出最大容量需要额外处理

```java
int newCapacity = oldCapacity + (oldCapacity >> 1);
```

### 2.2.2 Vector

Vector底层也是使用Object数组, 默认大小==10, **容量不足时扩容capacityIncrement(默认扩容两倍)**

```java
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
```

方法基本为同步方法, 线程安全

```java
    public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = obj;
    }
```

### 2.2.3 LinkedList

移除列表中的节点时, 需要将pre, item, next指针置空防止内存泄漏

原因: 节点虽然从LinkedList中删除, 但是如果可以通过其他方式可达该节点, 那么这个节点的next, item, pre也可达, 可能导致gc不能识别出垃圾

```java
        f.item = null;
        f.next = null; // help GC
```

## 2.3 HashSet&TreeSet
