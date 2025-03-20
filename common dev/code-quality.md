# 1. 术语

**_什么是测试驱动开发(TDD)?_**

```
先写测试代码, 再写业务代码
```

**_测试用例如何设计?_**

```
黑盒测试用例设计方法:
🌟 等价类划分
🌟 边界分析
```

**_什么是断言?_**

```
断言(assert)用于检查程序要正常运行必须要满足的条件, 只用在特殊的场景(如程序运行在太空中电位会可能发生跳变)

🌙一般情况下生产环境中不允许使用断言
```

# 2. 单元测试

**_为什么要进行单元测试?_**

```
BUG发现得越晚, 修复BUG的代价就越高
```

_**不同语言常用的单元测试框架?**_

| 语言/运行时  | 框架             |
| ------- | -------------- |
| Java    | Junit, Mockito |
| Cpp     | GoogleTest     |
| Python  | pytest         |
| Node.js | Jest, Vitest   |

_**在Junit的单元测试类中, 为什么没有main方法依然可以执行测试代码?**_

```java
🌟测试代码的主类由测试框架提供, 如Junit4的测试主类为JUnitCore(包含main方法)
🌟Maven/IDE会提供对应测试框架的测试主类(Maven的Surefire插件支持Junit等测试框架)
```

_**为什么不建议在单元测试中调用其他服务或访问数据库?**_

```
其他服务或者网络不稳定时, 可能造成测试失败或超时
```

**_Jacoco 等工具是如何统计单测行覆盖率的?_**

```
在编译后的字节码中插入指令, 效果等同于在每行代码前插入一行代码用于统计覆盖率
```

# 3.设计模式

**_说一下面向对象的设计原则?_**

```
🌟 开闭原则: 对扩展开放, 对修改关闭
🌟 合成复用原则: 组合优于继承
🌟 单一职责原则: 类的职责要单一
🌟 依赖倒置原则: 依赖于抽象编程
🌟 Demeter原则: 一个类不应该和其它不相关类有直接依赖
🌟 Liskov替换原则: 子类在任何地方都能替代父类
```

**_Lazy-Loading 的单例模式实现方式有哪些?_**

```
🌟 Double Check Lock
🌟 static 代码块
🌟 枚举类
```

**_代理模式和装饰器模式的区别?_**

```
代理模式一般只会代理一层, 装饰器模式一般封装多层
```

# 4. Java代码规范

## 1. Javadoc

**_javadoc 注释中常用的标签有哪些?_**

```
@param    参数描述
@return 返回值描述
@deprecated
@see 参考
{@link 包.类#成员} --文档内跳转
<p></p>
<a href="url">链接说明</a>

🌙使用javadoc可以生成代码文档
```

## 2. Lombok

**_lombok 中常用的注解有哪些?_**

| 注解                      | 说明        |
| ----------------------- | --------- |
| **@Getter**             |           |
| **@Setter**             |           |
| @ToString               |           |
| @EqualsAndHashCode      |           |
| **@XxxArgsConstructor** |           |
| **@Data**               |           |
| **@Builder**            | 支持链式调用初始化(注意会将构造函数私有化) |
| **@Slf4j**              | 支持 log    |

**_Lombok 注解如何生效?_**

```
🌙Java提供了SPI: AnnotationProcessor, 在编译阶段Java会调用该接口的实现类中的方法处理注解
```

## 3. 日志

**_常见日志框架和日志实现?_**

```
日志框架: Slf4j
日志实现: log4j/logback
```

**_常用日志的级别?_**

```
debug
info
warning
error
```

**_打印异常日志时需要记录哪些内容?_**

```
🌟 Exception对象(如有)
🌟 造成异常的业务对象
🌟 异常提示文本
```

## 4. 金额处理
***代码中金额用什么类型存储和计算?***
```
存储和加减乘计算使用Long, 除法计算使用BigDecimal.valueOf转换后进行计算(注意使用四舍五入/截断策略)
```

***为什么尽量使用BigDecimal.valueOf()方法而不是constructor方法来构造对象?***
```
new BigDecimal(double)可能造成精度丢失
```

***日期和金额在网络传输中使用什么类型?***
```
Long类型

🌙 JavaScript代码中的Number使用浮点数存储, 在(反)序列化时如果long过大会造成精度丢失!
```

# 5. Git

**_Git 和 SVN 的区别?_**

```
🌟 git是分布式架构, svn是cs架构

🌟 git对分支branch有更好的支持
```

**_git 常用指令?_**

```
git init
git add .
git commit -m "commit log"
git checkout 分支
git branch 新分支
git push
git pull
git fetch
git merge 分支
git cherry-pick commitId
git remote  add  <远程仓库名称例如origin>  <远程仓库URL>
git rebase --abort

git config --global user.name "用户名"
git config --global user.email "none@email.com"
git config --global http.proxy "http://localhost:7890"
```

 **_暂存区和工作目录的区别?_**

```
暂存区位于工作目录的.git/index文件中, 用于保存文件的本地修改记录
```

**_常用的开发分支?_**

```
master: 与线上代码相同
feature: 开发时从master分支拉取, 合并到master/release
release: 上线前将feature合并
hotfix: 线上修复时从master分支拉取, 最终会合并到master
```

**_.gitignore 文件的作用?_**

```
.gitignore中声明的文件(夹)不会被添加暂存区
```

**_Angular 规范?_**

```
<type>(范围): 描述

常用的type: fix, feat(大功能), impr, test, refactor, func(小功能)
```

**_git merge 和 git rebase 的区别?_**

```
git merge master: 将master上的新提交添加到当前分支

git rebase master: 将当前分支上的新提交添加到master
```

**_git revert 和 git reset 的区别?_**

```
revert 会有新提交, reset 不会有新提交
```

**_git cherrypick 的作用?_**

```
对单个commit进行合并
```

**_git 如何通过 ssh 公钥链接到远程仓库?_**

```
使用ssh-keygen生成公钥和私钥, 将公钥登记到远程仓库
```

# 6. 研发流程

**_讲一下研发的流程?_**

```
1. 用户提出TT
2. 产品根据TT提出需求
3. 需求评审
4. 技术方案评审
5. 编码
6. 自测
7. 前后端联调
8. 测试用例评审
9. 冒烟测试
10. 自动化测试
11. PR & Code Review
12. 线上灰度放量
```
