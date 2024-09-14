# 1. 术语

**_什么是测试驱动开发(TDD)?_**

```
TDD指的是 先编写测试代码 -> 然后写具体的代码实现 ->  最后在每次编译后都执行测试代码 这样的过程
```

**_测试用例如何设计?_**

```
* 黑盒测试中常常使用等价类+边界分析生成测试用例, 测试用例一般为6+N个, 上/下边界附近的各三个用例, N个正常用例

* 白盒测试注意分支覆盖和语句覆盖
```

**_什么是断言, 有什么注意事项?_**

```
断言(assert)用于检查程序要正常运行必须要满足的条件

注意: 生产环境中不允许使用断言
```

# 2. 单元测试

_**不同语言常用的单元测试框架?**_

| 语言/运行时 | 框架           |
| ----------- | -------------- |
| Java        | Junit, Meckito |
| Cpp         | GoogleTest     |
| Python      | pytest         |
| Node.js     | Jest, Vitest   |

_**加载测试类和执行测试的启动代码在哪里?**_

```java
// Junit加载测试类和执行测试的启动代码:
Result result = JUnitCore.runClasses(XxxTest.class); //JUnitCore是一个Runner
```

一般在构建工具中(如 Maven 的测试插件)或由 IDE 提供启动类(`com.intellij.rt.junit.JUnitStarter`)

_**为什么不建议在单元测试中调用其他服务或访问数据库?**_

其他服务或者网络不稳定时, 可能造成测试失败或超时

_**Junit 启动类的顶级类是什么?**_

Runner, 核心方法为 run(), SpringJUnit4ClassRunner 就是继承 Runner 从而扩展了测试, 在加载测试类之前启动 Spring 容器

_**Meckito 框架的最大优点?**_

不用启动 Spring 容器(太耗时), 自动生成和注入 Mock 对象和 Mock 方法返回值

**_Jacoco 等工具是如何统计单测行覆盖率的?_**

在编译后的字节码中插入指令, 统计代码是否执行到该位置

**_为什么没有 Assert 的单测意义不大?_**

没有 Assert 只能确保代码能执行完, 不能保证结果符合预期

**_测试用例如何编写?_**

编写测试用例至少包含 3 种用例:

```
* 正确输入 -> 正确输出
* 错误输入 -> 错误输出
* 边界正确输入 -> 正确输出
```

# 3.设计模式

**_说一下面向对象的设计原则?_**

```
* 开闭原则
* 合成复用原则: 组合优于继承
* 单一职责原则: 接口职责/类的职责要单一
* 依赖倒置原则: 依赖于接口/抽象编程
* Demeter原则
* liskov替换原则: 子类在任何地方都能替代父类
```

**_lazy-loading 的单例模式实现方式有哪些?_**

```
* Double Check Lock
* static 代码块
* 枚举类
```

**_代理模式和装饰器模式的区别?_**

```
代理模式一般只会代理一层, 装饰器模式一般封装多层
```

# 4. 代码规范

## 1. 通用代码规范

**_javadoc 注释中常用的标签有哪些?_**

```
@since <version>
----------
@param	参数说明
@return 返回值说明
@deprecated 过期说明
-----------
{@code 代码}	-- 代码块
{@link 包.类#成员}	--跳转
{@value 包.类#static成员}   --显示static变量的值
<br>
<p></p>
<a href="url">链接说明</a>
```

## 2. lombok

**_lombok 中常用的注解有哪些?_**

| 注解                    | 作用/使用示例               |
| ----------------------- | --------------------------- |
| **@Getter**             |                             |
| **@Setter**             | `@Setter(AccessLevel.NONE)` |
| @ToString               |                             |
| @EqualsAndHashCode      |                             |
| **@XxxArgsConstructor** |                             |
| **@Data**               |                             |
| **@Builder**            | 支持链式调用初始化          |
| **@Slf4j**              | 支持 log                    |

**_lombok 注解如何生效?_**

java 提供了接口 Annotation Processor, 在编译阶段 java 会调用该接口的实现类中的 process 方法处理注解, lombok 在此时修改抽象语法树

## 3. 日志

**_常见日志框架和日志实现?_**

日志框架: Slf4j

日志实现: log4j/logback

**_常用日志的级别?_**

debug/info/warning/error

**_打印异常日志时需要纪录哪些内容?_**

```
* Exception对象
* 造成异常的业务对象
* 异常提示文本

tip: 打印造成异常的业务对象时谨慎调用可能抛异常的方法
```

## 4. Git

**_Git 和 SVN 的区别?_**

```
* git是分布式架构, svn是cs架构

* git对分支branch有更好的支持
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
.gitignore文件中声明的文件(夹)在git add时不会跟踪
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

# 5. 研发流程

**_讲一下研发的流程?_**

```
1. 用户提出TT
2. 产品根据TT提出需求
3. 研发和产品进行需求评审
4. 研发根据提出技术方案
5. 研发编码
6. 研发自测
7. PR & Code Review
8. 测试人员测试
9. 灰度上线
10. 全量上线
```
