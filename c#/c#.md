# 1. C#简介

## 1. C#的编译和执行

_C#和.NET 的关系是什么?_

```
.NET是C#的运行时

tip: .NET平台可以运行C#, F#, VB程序, 提供GC
```

_C#源码被编译后会生成什么文件?_

```
>> C#源码被编译成IL语言

>> 程序入口的cs文件会生成exe文件

>> 其他cs文件会生成dll文件
```

_.NET Freamwork 和.NET CORE 的区别?_

```
.NET Freamwork只能依附Win平台, .NET CORE可以依附各种平台
```

## 2. C#项目管理

_.csproj 文件的作用是什么?_

```
.csproj 文件是C#项目说明文件, 相当于Maven中的pom.xml

tip1: 使用dot build指令可以读取.csproj并构建项目
tip2: 使用dot resotre可以下载csproj中声明的依赖的dll
```

_解决方案是什么?_

```
一个解决方案是一个C#项目集合, 使用一个.sln文件描述
```

_C#项目的发布方式有哪些?_

```
* 自包含发布(内置.NET运行时)
>> dotnet publish -r win-x64 -c Release --self-contained

* 直接发布(不包含.NET运行时)
>> dotnet publish -r win-x64 -c Release

* 单文件发布(发布为单个exe文件)
>> dotnet publish -r win-x64 -c Release --self-contained -p:PublishSingleFile=true

```

# 2. 语言特性

## 1. 语法特性

_var 关键字和 dynamic 关键字的区别?_

```
var会在编译时推断出具体类型, dynamic相当于Object
```

_C#中如何声明和使用一个二位数组?_

```csharp
int[,] arr = new int[3, 4];
arr[1, 2] = 5;
```

## 2. 委托和方法

_方法参数中的 out | in | params | ref 关键字的作用?_

## 3. 面向对象

_C#中的属性是什么?_

## 4. 迭代器

# 3. 集合

# 4. API

## 1. System

_标准库在哪个命名空间下?_

```
在System下

tip: 默认情况下cs文件隐式using System;
```

_C#中包装类和内置类的关系是什么?_

```
内置类型(如int)和包装类(如Int32)实质上共用一个Class元信息
```

# 5. 异步编程
