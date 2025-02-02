# 1. Maven

## 1. Maven 简介

_在不使用构建工具的情况下如何创建Java程序包?_

```
1️⃣使用javac编译出所有class文件
2️⃣使用jar打包class文件, 资源文件以及第三方jar包
3️⃣使用java运行jar包和测试jar包
4️⃣将jar包发送并部署到服务器
```

_怎样理解 Maven 中的约定大于配置(Convention Over Configuration)?_

```
maven程序中的属性(System.getProperties() + 应用属性)都有默认值, 可以在super pom中可以查看
```

_怎样统一个项目使用的 maven 版本?_

```
1️⃣在项目中使用mvn -N wrapper:wrapper生成wrapper相关文件

2️⃣使用mvnw替代mvn指令
```

_如何使用maven创建一个空项目?_

```
mvn archetype:generate

🌙archetype为maven自带插件
```

## 2. POM

_怎么标识唯一的项目?_
```
GAV坐标
```

_讲一下 POM 的继承体系?_

```
Super POM → Parent POM → Child POM

父子项目通过pom中标签标记:
🌙子项目pom中有<parent>
🌙父项目pom中有<package>pom<package>和<modules>
```

_maven 插件中的 goal 是什么_

```
一个goal对应插件的一个指令, 在<execution>标签中绑定phase和插件goal

🌙使用mvn <phase>可以按顺序执行到特定阶段(该阶段及之前阶段的goal都会执行)
🌙使用mvn <plugin>:<goal>执行特定goal
🌙maven插件本质也是一个jar包, 因为maven运行在jvm中
```

_讲一下 Maven 中 Build Lifecycle 中主要的 phase?_

```
🌟clean
🌟compile
🌟test
🌟package
🌟install: 安装到本地仓库

🌙所有phase MAVEN都提供默认的插件
```

_profile标签的作用?_

```
profile标签中的配置只有在满足profile的条件时才会生效
```

## 3. 依赖管理

_maven中依赖项的_`<scope>`_有哪些?_

```
🌟 complie(默认): 用于正式程序jar的打包
🌟 test: 用于测试程序jar的打包, 如Junit
🌟 provided: 运行环境自动提供, 仅用于编译, 不会被打包到jar中, 如Servlet
🌟 runtime: 编译不需要, 仅用于运行, 会被打包到jar包中, 如Mysql-Connector-J
```

_版本号_ _`1.0-SNAPSHOT`_ _1.0-SNAPSHOT_ _中的 SNAPSHOT 有什么含义?_

```
maven在每次编译时会尝试从仓库中获取并使用最新子版本
```

_什么是依赖冲突, 如何解决?_

```
两个版本的依赖都存在时, maven可能会选择低版本的依赖, 导致不能使用高版本特性

🌙使用<dependancyManagement>锁定依赖为高版本
```

_如何配置远程仓库地址&插件地址和代理策略?_

```
在Maven安装目录/conf/settings.xml中配置
```

# 2. Gradle

## 1. Gradle简介

_Gradle Wrapper 是什么?_

```
为了统一项目使用的Gradle版本, 使用项目文件夹中的Gradle Wrapper代替Gradle

🌙使用gradlew指令替代gradle
```

_如何新建一个 gradle 结构的项目?_

```
gradle init
```

_Gradle的依赖项会被保存在哪里?_

```
默认保存在全局缓存目录~/.gradle中

🌙Gradle依赖项和Maven相同, 都使用GAV坐标
```

_Gradle的编译输出会被保存在哪里?_
```
默认保存在项目文件夹下的build目录
```

_Gradle 为什么要使用 Client+Deamon 的方式?_

```
避免Deamon JVM的冷启动, 每执行一个指令仅仅启动Client JVM
```

_Gradle 的生命周期分为哪三步?_

```
1️⃣Initialization: 执行settings.gradle代码, 指定项目模块等基本信息

2️⃣Configuration: 执行build.gradle代码, 构建Project对象和Task对象

3️⃣Execution: 执行Task和Project中定义的回调
```

_Gradle是如何实现Incremental Build的?_
```
每个Task都需要明确定义输入和输出(文件/文件夹), 在执行Task前会对检查输入文件的SHA-1码, 如果输入相同, 则跳过该Task的执行
```


## 2. Groovy简介

_Groovy 和 Java 语言的相同点和区别?_

```
🌟两者代码都会被编译为字节码运行在JVM上
🌟Groovy有更多的语法糖
```


## 3. Gradle API

_如何执行Gradler任务?_

```
使用指令gradlew (:子项目name:)<task名称>可以执行任务

🌙使用gradle tasks查看所有任务
🌙一个任务可以依赖其他任务(Task#dependsOn方法)
```

_build.gradle 中调用的方法来自于哪一个类?_

```
Project类
```

_Gradle中的插件是什么?_

```
插件类实现了org.gradle.api.Plugin, 会被打包成jar

🌙 Gradle默认提供了一些插件(如Java插件)
```

# 3. CMake
