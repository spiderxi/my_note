# 1. 常识

**Python和C++/Java的主要区别？**
```
🌟C++/Java是强类型语言， Python不是
🌟Java编译成字节码运行在JVM并支持JIT， Python直接逐条被解释器并执行
🌟C++手动管理内存， Java和Python都支持GC

🌙JVM/Python解释器底层都是调用C++函数和系统调用API
```

**python和pip和conda的区别？**
```
🌟原生python仅包含标准库
🌟pip和conda两者都为python包管理工具， conda额外支持环境管理

🌙 miniconda = conda + 一些预装包
```

**什么是python包？**
```
🌟包一般一系列.py文件通过import到python代码中使用
🌟复杂包可以是一个程序, 对于多数程序pip安装后会自动添加到path
```

**什么是venv？**
```
一个venv中包含独立：
🌟python解释器
🌟三方包目录
🌟包管理工具
🌟环境变量

🌙 项目级的环境变量可以通过创建.env文件来设置
```

****

