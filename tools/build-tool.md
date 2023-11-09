# 1. Maven

## 1. 简介

***为什么你要使用Maven?***

```
如果不使用Maven, 需要手动使用javac编译所有class文件并使用jar打包class文件和资源文件, 使用java运行jar包和测试
```

***怎样理解Maven中的约定大于配置(Convention Over Configuration)?***

maven中使用的属性值都有默认值(例如源码路径为 `src/main/java`)

![1696733941119](image/build-tool/1696733941119.png)

***怎样统一个项目中所有人员使用的maven版本?***

```
1. 在项目中使用mvn -N wrapper:wrapper 生成 mvnw.cmd/mvnw 批处理文件

2. 使用 mvnc.cmd 替代 mvc 指令
```

## 2. POM

***怎么标识唯一的项目?***

使用 `GAV` 坐标

***讲一下POM的继承体系?***

```
* 所有POM继承自Super POM, Super POM中包含maven约定的配置
* 子POM可以覆盖父POM中的配置
```

***讲一下Build Lifecycle中主要的phase?***

```
* clean
* compile
* test
* package
* install
```

***定义插件时goals和phase有什么用?***

```
* goals指插件内部的指令
* phase指插件关注的声明周期阶段

使用 mvc <phase> 或 mvn <plugin>:<goal>执行插件
```

***profile的作用?***

```
profile中的配置只有在满足profile的条件时才会生效
```

> 当多个profile同时生效时, 会合并profile, 按照 ` pom.xml > settings.xml; 后 > 前`的优先级覆盖

## 3. 依赖管理

***依赖的  `<scope>` 有哪些?***

```
* complie/default: (编译需要的依赖)
* runtime: 运行需要的依赖
* test: 测试需要的依赖
* optional: 无依赖传递性的依赖
* provided: 运行环境提供的依赖
```

***版本号 `1.0-SNAPSHOT` 中的SNAPSHOT有什么含义?***

`maven在每次编译时, 会尝试获取并使用最新子版本`

***如何添加文件系统中的依赖?***

`将依赖的scope设置为system并指定路径`

```xml
<scope>system</scope>
<systemPath>${basedir}\src\lib\ldapjdk.jar</systemPath>
```

***什么是依赖冲突, 如何解决?***

```
* 依赖冲突: 两个依赖项的GA相同但Version不同时, maven可能会选择低版本的依赖, 此时可能导致依赖
另一个高版本的其他依赖项不能正常使用

>>> 解决方法: exclue(排除依赖) / dependancyManagement(版本锁定)
```

***maven查找依赖的顺序?***

`local repository  =>  mirror  =>  remote repositroy(such as the central)`


# 2. Webpack
