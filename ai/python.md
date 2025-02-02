# 1\. 简介

## 1\. 开发环境

_**什么是 conda?**_

```
管理python环境(python版本 + package集合)的工具, 可以在多个python环境下快速切换

Tips:
相关指令: 1️⃣conda init(将 Conda 的启动脚本添加到 Shell 配置文件) 2️⃣conda create --name myenv python=3.9 3️⃣conda activate myenv(将env脚本对应的目录添加到临时shell的path中)
```

**_什么是 pip?_**

```
python自带的一个package, 用于从官方仓库中下载package到本地python的lib目录下

Tips:
相关指令: 1️⃣pip install 2️⃣pip list
```

**_python package 项目中应该包含哪些内容?_**

```
1️⃣ 模块代码文件.py 2️⃣ 资源/配置文件 3️⃣ setup.py

Tips:
在setup.py中可以声明entry_points将命令行命令映射为python模块的函数(pip在安装package时生成命令的可执行文件)
```

***什么是CUDA?***
