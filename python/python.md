# 1. 常识

**Python和C++/Java的主要区别？**
```
🌟C++/Java是强类型语言， Python是动态类型语言
🌟Java编译成字节码运行在JVM并支持JIT， Python直接逐条被解释器并执行
🌟C++手动管理内存， Java和Python都支持GC
🌙Python缩进是语法的一部分, C++/Java不是

🌙JVM/Python解释器底层都是调用C++函数和系统调用API
```

**python和pip和conda的区别？**
```
🌟原生python仅包含标准库
🌟pip和conda两者都为python包管理工具， conda额外支持环境(venv)管理

🌙 miniconda = conda + 一些预装包
🌙 pip本身也是一个包, 并python原生自带pip
```

**什么是python包？**
```
🌟包一般为一系列.py文件, 通过import到python代码中使用
🌟复杂包可以是一个程序, 对于多数程序pip安装后会自动添加到path
```

**什么是venv？**
```
一个venv中包含独立：
🌟python解释器
🌟三方包目录
🌟包管理工具
🌟环境变量

🌙 实际项目中, 环境变量一般通过xxx.env文件保存, 并通过dotenv库加载
```

# 2.语言特性

## 继承体系

**python中所有类的基类是啥?**
```
object

🌙 内置函数id(obj)可以获取对象地址
🌙 使用内置函数type(obj) isinstance(obj, ClassName)检查类型
🌙 type(obj) 的返回值本身也是"type"类的实例
```

**==和is比较的是值还是地址?**
```
🌟 ==比较的是值, 等价于obj.__eq__()方法
🌟 is比较的是地址
```

**什么是鸭子类型（Duck Typing）?**
```
python设计哲学: 如果它走起来像鸭子，叫起来像鸭子，那它就是鸭子
即: 只关心对象(obj)的行为(method), 不关心对象的类型(type)

🌙 运行时通过hasattr()判断, 而不是isinstance()判断
🌙 with-as代码块本质上判断了obj有无__enter__()和__exit__()两个方法, 有就调用
```

**什么是MRO?**
```
python支持多继承, 使用MRO(method resolve order)解决method冲突问题, 使用C3 线性化算法排序父类决定优先级

🌙 MRO存储在obj.__mro__
```

## magic method/variable

**常见的内置object magic method有哪些?**
```
🌟 对象生命周期相关: __init__()
🌟 属性访问控制: __setattr__()  __getattribute__() __getattr__()
🌟 运算符重载: __add__() __eq__()
🌟 duck typing相关: __str__() __call__() __hash__()

🌙 获取属性时优先调用__getattribute__(), __getattr__()仅在属性不存在时调用, 动态返回属性值
```

**常见的内置magic variable有哪些?**
```
🌟 __name__: 模块名/类名/方法名
🌟 __doc__: 代码中的文档注释
🌟 __class__: obj.__class__返回当前对象的type
🌟 __bases__: obj.__bases__返回父类元组
🌟 __dict__: obj.__dict__返回一个字典, key=属性名, value=值, 可动态修改
```

**为什么hasattr(obj, "method")返回True?**
```
方法本质上是类的属性，其值是一个函数对象("method"实例)

🌙 类内部的方法type=method, 外部定义的函数type=function
```

**python中的反射机制是怎么实现的?**
```
对象内部维护magic variable, 可动态读取/修改

🌙 实际反射时不用直接读写magic variable, 提供了内置方法如getattr(p, "name"), 底层调用__getattribute__(), 最终从__dict__读取
```

## 异常处理
**python所有异常的基类是什么?**
```
Exception
```

**异常处理相关代码语法?**
```
🌟 异常处理代码块: try-except-finally
🌟 抛出异常: raise
🌟 资源自动关闭: with-as
```

## 函数与装饰器
**装饰器的核心思想是什么?**
```
装饰method/type, 返回一个增强版本

🌙 使用@Xxx来使用装饰器Xxx
🌙 装饰器装饰method可实现aop
🌙 装饰器本身也可以被装饰器装饰
```

**装饰器的本质是什么?**
```
装饰器本质上是一个function, 这个function入参为function/type, 内部定义一个增强后的function/type, 然后返回

🌙 语法糖"@decorator"装饰"def func()"时, 可以理解为先def func(), 然后func = decorator(func)
```

**函数的各种传参方式的声明顺序?**
```
位置参数 > 缺省参数 > 不定参数 > 关键字参数

🌙 位置参数声明: def func(a, b)
🌙 缺省参数声明: def func(a, b=1)
🌙 不定参数声明: def func(a, *args), args本质是一个元组
🌙 关键字参数声明: def func(a, *args, **kwargs), kwargs本质是一个字典
🌙 即使函数声明为位置参数, 调用时仍然可以使用关键字参数传参, 会自动匹配到位置参数, 例如func(1, b=2, c=3)的方式调用def func(a, b, **kwargs), 最终a=1, b=2, kwargs={"c": 3}
```

**lambda表达式是什么**
```
快速声明函数变量的一种方式

🌙 例如func = lambda 参数1,参数2:表达式
```

**什么是函数闭包(Closure)?**
```
即使外部函数已经执行完毕, 内部函数也可以访问外部函数的变量

🌙 闭包原理: 内部函数对象的__closure__存储了外部函数的变量
```

# 3. 标准库
## 内置(built-in)模块
**python中常用的内置类型有哪些?**
```
🌟 int float bool None
🌟 list str set
🌟 tuple dict

🌙 初始化时 1 = int类型 1.0 = float类型
🌙 初始化时 [1, 2, ""] = list类型 {1, 2} = set类型
🌙 初始化时 1,2 = tuple类型 {"name": "klein"} = dict类型
🌙 python中所有类型都继承自object, 包括内置类型
```

## 模块
**python常用的内置模块?**
```
🌟 系统交互类: os sys
🌟 工具类: json collections math random datetime re
```

