# 1. 项目管理

## 1. 项目管理文件

_一个项目如何统一代码风格?_

```
使用.editorconfig文件/.prettierrc文件配置代码风格, 该文件可以被多数IDE识别并生效(可能需要额外插件)
```

_常用的配置文件的格式有哪些?_

```
🌟xml文件
🌟properties文件
🌟yml文件

🌙Java中可以直接使用java.util.Property解析properties文件
🌙可以导入xsd文件规定xml文件的结构/默认值/约束值
🌙${Xxx}等占位符需要具体应用进一步解析, 工具类一般不提供支持
```

_项目中常见的文件?_

```
README.md: 项目简介
licsence.md: 开源协议
```

## 2. 平台兼容

_LF CRLF的区别是什么?_

```
🌟 unix系统使用LF LF=\n
🌟 windows系统使用CLRF CLRF=\r\n
```

_环境变量是什么?_

```
环境变量是由操作系统维护的变量, 进程(如jvm, shell)可以读写环境变量

🌙环境变量PATH作用: Shell中会在PATH目录下搜索文件
```

_win 和 linux 系统如何管理软件的安装和卸载?_

```
🌟 linux: 使用包管理器(如apt yum)
🌟 win: 使用installer程序进行安装(基于.NET平台, 安装的同时可修改注册表)

🌙: 安装便携式应用可以直接解压程序文件到文件系统中
```

## 3. 开源协议

_什么是开源协议?_

```
使用开源代码时需要遵循的法律条款
```

_常见开源协议的特点?_

```
🌟 MIT: 保留源代码中的版权声明和License文件, 不能因为代码问题起诉作者
🌟 Apache: 在MIT的基础上如果对代码修改了需要添加修改说明
🌟 GPL: 必须以GPL协议开源自己的源码
```

# 2. 代码常识

## 1. RegExp

_如何匹配数字/字符串开始/字符串结束/单词边界/空白符/非空白符/任意字符?_

```
🌟 十进制数字: \d (decimal)
🌟 开始和结束: ^这是一段文本$
🌟 单词边界: \b
🌟 空白符: \s
🌟 非空白符: \S
🌟 除换行外的任意字符: .
```

_如何匹配特定数量的字符?_

```
🌟 零次或一次: ?
🌟 一次及以上: +
🌟 固定次数: {n}
🌟 次数区间: {n, m}
```

_正则表达式中如何匹配左括号?_

```
\(
```

## 2. Cron 表达式

_Cron 表达式是什么?_

```
一个Cron表达式定义了一个定时任务的执行时机
```

## 3. URI

_linux 中怎么表示上级目录和当前目录以及用户目录和根目录?_

```
🌟当前工作目录 = .
🌟工作目录的上级目录 = ..
🌟用户目录 = ~
🌟根目录 = /
```

_Apache Ant 风格的路径匹配规则是怎样的?_

```
🌟使用*匹配不含路径分隔符"/"的字符串
🌟使用**匹配包含路径分隔符"/"的字符串
```

## 4. MIME

_MIME 文件类型由哪三部分组成?_

```
由"主类型+子类型+参数列表"组成, 例如:
type/subtype; param1=value1; param2=value2
```

_HTTP提交表单时如果表单中包含文件那么Content-Type是什么?_
```
Content-Type: multipart/form-data; boundary=参数分隔符

🌙文件被二进制编码后作为表单的一个参数
```

_如何设置 Http 响应头让浏览器将响应体下载为附件?_

```
Content-Disposition: attachment; filename="demo.pdf"
```

# 3. 生活常识

## 1. 时间

*时间戳和本地时间的区别是什么?*
```
🌟 全球任何地方任何一刻, 时间戳的值都相等(GMT时间1970年1月1日00:00.000对应的时间戳为0)

🌟 同一个时间戳, 不同时区的本地时间字符串表示不同(如上海8:00AM, 纽约8:00PM)
```

*UTC GMT CST 时间的区别是什么?*
```
🌟 GMT(Greenwich Mean Time)基于天文现象, UTC(Coordinated Universal Time)基于原子钟, 同一时刻两者时间戳相差900左右

🌟 CST(China Standard Time)等价于UTC+8
```

*计算机时间戳同步协议NTP是如何消除网络传输的延迟的?*
```
利用本地时间戳diff和时间服务器上的时间戳diff估计网络耗时
```
![alt text](image/extra/image.png)
# 4. 计算机系统常识

## 1. 硬件

_存储单位从小到大有哪些?_

```
8 bit (b)
= 1 byte (B)
= 2^-10 Kibibyte (KiB)
= 2^-20 Mebibyte (MiB)
= 2^-30 Gibibyte (GiB)
= 2^-40 Tebibyte (TiB)
= 2^-50 Pebibyte (PiB)

🌙ISP经常将十进制单位和二进制单位混淆用来营销, 十进制的单位有:
Kilobyte (KB)
Megabyte (MB)
Gigabyte (GB)
Terabyte (TB)
Petabyte (PB)
```

_计算机中各种基本操作的时间大概为多少?_

| 操作                  | 时间                      |
| --------------------- | ------------------------- |
| 执行一个指令          | 1 ns                      |
| L1-L3 cache 访问     | L1 < 1 ns<br />L3 = 50 ns |
| Memory 访问           | 100 ns                    |
| SSD 随机访问/顺序访问 | 0.1 ms                    |
| HDD 顺序访问          | 2ms                       |
| HDD 随机访问          | > 10ms                    |
| 北京到上海网络延迟    | 30ms                      |
