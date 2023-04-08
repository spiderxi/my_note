# 0. Linux与硬件

## 0.1 CPU架构

RISC指令集下的CPU架构包括

```
ARM, PowerPC, SPARC
```

CISC指令集下的CPU架构包括

```
x86, x86_64
```

## 0.2 主板和CPU

主板上包含各种硬件设备的插槽, 主板用于连接各个硬件设备(通过总线/), 各个设备的接口符合特定的接口标准

ASUS Asus Z97-AR主板的各个硬件接口标准

```
CPU （Intel LGA 1150 Socket）
内存 （DDR3 3200 support）
显卡接口 （PCIe3.0）
SATA 磁盘插槽 （SATA express）
```

主板上会有南桥芯片用于数据在传输过程的缓冲

```
南桥负责速度慢的设备(鼠标键盘, 外存, 网卡)的缓冲
现代CPU直接与主存/显卡沟通, 不需要外部芯片做缓冲(以前有北桥)
```

CPU频率

```
外频: CPU与其他设备进行数据传输的频率
内频: CPU内部使用的时钟频率, 可能会动态变化以节省电量
```

## 0.3 显卡和GPU

显卡

```
显卡又称为VGA（Video Graphics Array）, 用于要输出存储图像
1024x768分辨率的全彩显示（每个像素占用3Bytes的容量）需要的显卡容量至少为1024*768*3Byte
实际容量还要考虑屏幕刷新率, 屏幕刷新率高的话需要显卡容量更大
```

GPU

```
集成在显卡内部的计算芯片, 用于3d图像的运算, 避免CPU计算繁忙
```

显卡与输出设备的接口标准

```
D-Sub
DVI
HDMI
Display port
```

## 0.4 外存类型和接口

外存分为SSD(固态硬盘)和HDD(机械硬盘)

```
SSD: 相当于大容量flash,容量较小, 速度快, 寿命短
HDD: 内部使用磁盘+磁头, 容量大, 速度慢, 寿命长
```

外存接口标准

```
SATA SATA3速度=600MB/s
SAS SAS3速度=1200MB/s
USB USB3.0速度=500MB/s
PCIe PCIe2.0速度=500MB/s
```

## 0.5 BIOS和CMOS

BIOS

```
存储在主板上EEPROM的一个微型操作系统, 用于开启启动后检查硬件和加载操作系统
```

CMOS

```
主板上存储硬件信息和系统时间等信息的RAM芯片, 使用主板上的纽扣电池供电
纽扣电池没电时会使用出厂信息
```

## 0.6 磁盘分区

MBR分区方式中, 第一个分区包含部分boot loader程序和分区表, 第一个分区的大小是512Byte

# 1. 入门指令和快捷键

入门指令

```
ls -al
cd 
pwd
date +%H:%M:%S
cal 2023
bc
locale
man date
whatis man
sync
shutdown
```

快捷键

```
ctrl+c: 中断程序
ctrl+d: EOF
table table: 补全命令/参数/文件
shift+pageUp: shell翻页
↑/↓: 上一个指令, 下一个指令
```

# 2. 文件管理

## 2.1 文件权限

`ls -al`查看文件详细信息

```
-rwxr-xr-x  1 root root 15960 Apr  4 10:38 a.out*
文件类型+创建者权限+创建者所属群组权限+其他人权限+硬链接数+创建者+创建者群组+大小+最后修改时间+文件名
```

改变文件的权限

```
chgrp users a.out
chown root a.out
chmod a=rwx a.out
```

文件夹的权限

```
r代表这个文件夹可以使用ls查看目录结构
w代表可以往文件夹中添加/删除(不论要删除的文件是否有w权限)文件或目录
x代表可以作为工作目录
```

## 2.2 FHS

目录树

```
/bin 一般用户可以使用的可执行文件
/boot 开机会使用的文件
/dev 硬件设备对应的文件
/etc 操作系统配置文件
/lib 静态or共享函数库
/media /mnt 被挂载的设备所在的目录
/tmp 临时文件存放目录
/opt 第三方软件所在目录
/home 普通用户个人目录
/root root用户个人目录
/proc /sys 虚拟目录, 分别存储进程信息和系统信息
/usr unix software resource 所有用户的共享文件夹
/var 软件或系统运行时产生的文件
```

## 2.3 文件和文件夹操作

文件夹管理

```
cd ~
pwd
ls -al
mkdir -p test/inner
rm -r test
```

文件管理

```
cp -r test /tmp/test
mv test /tmp/test
rm -ir test
touch helloworld
ln -s /bin /root/bin
```

查看文件

```
cat /etc/hosts
less /etc/hosts
vim /etc/hosts
head --lines=3 /etc/hosts
tail --lines=3 /etc/hosts
grep pattern test.txt
```

文件默认权限和隐藏权限

```
umask -S
chattr +ai hello
lsattr hello
```

文件的查找

```
type ifconfig
which ifconfig
find /root -name hello
```

## 2.4 磁盘管理

查看个分区情况和挂载点

```
df -h
```

## 2.5 文件压缩

gzip只能对单个文件进行压缩, tar可以将文件夹合并成一个大文件

```
gzip hello
gzip --decompress hello.gz

tar -czf exp.tar.gz exp
tar -xzf exp.tar.gz -C ./
```

# 3. Vim

常用按键

```
i进入插入模式 
esc退出插入模式
:wq写入并退出
/ ?:向上向下查找
2dd: 删除从光标所在行开始的2行
2yy: 复制从光标所在行开始的2行
p: 复制到光标所在的下一行
:set nu 显示行号
0 移动到行首
$ 移动到行尾
```

# 4. Shell

## 4.1 bash

linux默认的shell是bash, 在/etc/passwd文件中可以查看用户登录后使用的shell

bash的优点

```
支持通配符  如ls /usr/Test*
可以使用历史命令  如↑↓
tab自动补全
支持alias别名
```

## 4.2 变量

内置变量

```
PATH=环境变量 
HOME=用户的主目录
```

变量的使用/添加/删除

```
echo ${PATH}
PS1="O(∩_∩)O: "
unset NAME
```

自定义变量只能在当前bash中生效, 如果想在子程序中生效可以使用export导出为环境变量

```
export NAME
```

查看变量

```
所有环境变量: env
所有变量: set
```

提示用户输入变量

```
read -p "请输入在30s内你的名字:" -t 30 NAME
```

声明一个整数类型变量

```
declare -i age
age=18+8
echo $age //26, 如果不声明变量为整数类型将会使用默认的字符串类型
```

## 4.4 别名和历史

给命令设置别名

```
alias la="ls -a'
```

查看history和执行上一个命令

```
history
!!
```

## 4.5 配置shell和自定义

bash分为login shell(使用shell时需要输入账号密码登录)和non-login shell, 两种shell的配置方式不同

login shell 配置文件`~/.bash_profile`

non-login shell配置文件`~/.bashrc`

通过配置文件可以设置shell的默认环境变量等信息

```bash
else
 PS1="O(∩_∩)O: "
```

## 4.6 重定向和管道

**将标准输出重定向**到文件或设备 /  将文件或设备作为标准输入

```
ls > test.txt  //覆盖
ls >> test.txt //追加
sort < test.txt //文件或设备作为输入
```

**管道符:** 将一个程序标准输出作为另外一个程序要输入的文件(此时省略文件名参数)

```
ls | grep hello
ls | grep "t[ae]st"
echo "hello" | grep el
echo "hello:name" | cut -d : -f 1    (输出hello)
```

常用于管道符的程序

```
grep(查找符合RE的行)
sort(按行进行排序) 
uniq(去除重复的行)
cut(对每一行按分隔符切割后取出分割后的内容)
```

$() 让指令输出作为**占位符**的内容

```
echo "当前linux内核版本为: $(uname -r)"
```

**特殊的文件:** *-表示标准输入输出 /dev/null表示黑洞文件*

```
echo "shit" > /dev/null

```

## 4.7 指令编程

使用逻辑运算&& ||可以实现程序间的条件执行, 如查看文件是否存在, 如果存在则删除

```
ls ./hello && rm ./hello
```

使用printf可以打印格式化字符串

```
printf "当前内核版本: %s \n" $(uname -r)
```

使用awk实现条件循环
