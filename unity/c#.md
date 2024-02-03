# 1. 基础

## 1. C#的编译和执行

**_C#和.NET 的关系是什么?_**

```
.NET是C#的运行时, C#源码被编译成IL语言后在.NET中执行
```

**_C#源码被编译后会生成什么文件?_**

```
>> 程序入口的cs文件会生成exe文件(该文件运行需要dll文件和.NET运行时)

>> 其他cs文件会生成dll文件
```

**_.NET 平台的特点?_**

```
>> 提供GC

>> 可以运行C#, F#, VB等语言写的程序
```

**_.NET Freamwork 和.NET CORE 的区别?_**

```
>> 两者都属于.NET运行时

>> .NET Freamwork只能依附Win平台, .NET CORE可以依附各种平台
```

## 2. C#项目管理

**_.csproj 文件的作用是什么?_**

```
>> .csproj 文件是C#项目说明文件, 包含项目元信息, 项目的依赖, 项目的构建配置属性组

>> 使用dotnet创建项目模板时自动生成.csproj 文件
```

**_MSBuild 是什么?_**

```
.Net SDK中的项目构建工具, 使用dot build调用MSBuild读取.csproj文件并构建项目
```

**_解决方案是什么?_**

```
一个解决方案是一个C#集合, 解决方案的元信息存储在.sln文件中
```

**_如何对 C#项目进行依赖管理?_**

```
>> 直接将依赖的dll文件下载到本地并在.csproj中声明

>> 使用NuGet进行包管理, 在.csproj声明依赖项目后使用命令"dot resotre"下载依赖
```

**_C#项目的发布方式有哪些?_**

```
>> 自包含发布(内置.NET运行时)
   	dotnet publish -r win-x64 -c Release --self-contained

>> 直接发布(需要执行程序的机器上有.NET运行时)
	dotnet publish -r win-x64 -c Release

>> 单文件发布
	dotnet publish -r win-x64 -c Release --self-contained -p:PublishSingleFile=true

```

# 2. 语言特性

## 1. 语法特性

**_var 关键字和 dynamic 关键字的区别?_**

```
var会在编译时推断出具体类型, dynamic相当于Object
```

**_C#中如何声明和使用一个二位数组?_**

```csharp
int[,] arr = new int[3, 4];
arr[1, 2] = 5;
```

## 2. 委托和方法

## 3. 面向对象

## 4. 迭代器

# 3. 集合

# 4. .NET API

## 1. System

**_System 命名空间下有些常用的类?_**

```
>> Console: 控制台交互类

>> Math, Random, File, DateTime等工具类

>> Array, String, Int32等数据类型/容器类

tips: 默认情况下cs文件隐式using System;
```

**_C#中内置类及其对应的包装类是什么? 包装类和内置类的关系是什么?_**

| 内置类型                          | 包装类                                  |
| --------------------------------- | --------------------------------------- |
| short & int & long & uint & ulong | Int16 & Int32 & Int64 & UInt32 & UInt64 |
| char                              | Char                                    |
| double & float                    | Double & Float                          |
| bool                              | Boolean                                 |
| string                            | String                                  |

```
c#中, 内置类型和包装类实质上共用一个元Class信息
```

# 5. 异步编程
