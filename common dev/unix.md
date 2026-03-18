# 1.简介
## 名词
*如何和系统内核交互?*
```
使用系统调用(system call)
```

*shell:应用程序:公共函数库:内核:系统调用之间的关系?*
![alt text](image/unix/image.png)

*文件描述符可以表示什么?*
```
🌟 一个进程已打开的文件(映射为VFS中的inode)
🌟 一个socket

🌙 使用shell创建子进程时, 会创建三个文件描述符(标准输入/输出/错误流, 默认指向键盘文件/终端文件)
```

*VFS(Virtual File System)的作用?*
```
在操作系统内核和外部IO设备之间添加了一个抽象层
```
![alt text](image/unix/image-1.png)

*POSIX标准主要定义了Unix系统的哪些内容?*
```
🌟系统调用
🌟文件系统
🌟进程/线程模型
🌟shell工具及其相关实用程序(ls/mv/grep)
```

## 硬件
_x86 和 arm 架构的区别?_
```
🌟x86 CPU: 使用复杂指令集, 例如intel芯片电脑
🌟arm CPU: 使用精简指令集, 例如mac电脑

🌙 CPU架构不同, 同一个指令(如加法)对应的二进制机器码指令也不同
🌙 如果涉及在x86机器上编译arm架构的cpp程序/操作系统时, 需要使用交叉编译器并指定sysroot(不能使用自带的编译器)
```

_什么是 BIOS 和 CMOS?_
```
🌟BIOS: 主板上的BIOS(Basic Input/Output System)是一个ROM, 包含一个程序, 作用是初始化硬件和引导操作系统
🌟CMOS: 主板上的CMOS是一个RAM, 存储BIOS配置(如系统时间戳)
```


