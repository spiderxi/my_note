# 1. Docker 简介

## 1. 术语

_什么是 DevOps?_

```
DevpOps = Development & Operation
DevOps: 从构建到部署和运维这一套开发流程是自动化的和可持续的
```

_IaaS, PaaS, Saas 的区别?_

```
* IaaS: 云服务商提供硬件资源, 包括存储器, 网络, CPU
* PaaS: 云服务商提供应用的运行环境, 包括操作系统,运行时,数据库软件等
* SaaS: 云服务商提供基础的应用服务(例如DBMS), 可以通过网络访问这些服务来构建自己的应用
```

_Docker 采用什么架构?_

```
* 采用CS + Registry架构, Registry上保存镜像, Server上运行多个容器
```

_什么是 Image, Dockerfile, Container?_

```
* 镜像: 容器的模板, 包含操作系统,文件系统,代码库等
* Dockerfile: 镜像的定义文件
* Container: 运行中的镜像, 一个镜像可以对应多个运行中容器, 各个容器的文件系统隔离
```

## 2. 基本使用

**_如何创建镜像并启动一个 docker 容器?_**

```
* 在项目文件夹下创建Dockerfile
* docker build -t <镜像名称> <项目文件夹路径>
* docker run -dp <宿主机的端口号>:<容器的端口号> <镜像名称>
```

启动容器成功后使用 `docker ps (-a)` 查看容器信息

**_Dockerfile 常用关键字有哪些?_**

```Dockerfile
FROM <基础镜像名称>
WORKDIR <工作目录>
COPY <构建环境中的目录> <容器中的目录>
RUN <镜像构建成功后执行的shell命令>
CMD <容器启动成功后需要执行的shell命令>
```

**_docker rm 和 docker stop 的区别?_**

```
* docker stop <容器id>: 日志/缓存文件会保留, 使用docker start <容器id>可以再次运行

* docker rm -f <容器id>: 停止容器运行并删除容器
```

## 3. 进阶使用

**_如何向 docker 仓库推送和拉取镜像?_**

```shell
docker login # 登录docker registry
docker tag <本地镜像名> <远程镜像名>
docker push <远程镜像名>

docker pull <远程镜像名>
```

> Docker 默认的远程镜像仓库为[dockerhub.com](https://hub.docker.com/), 通过修改配置文件可以修改远程仓库

**_绑定挂载(bind mount)和卷挂载(volumn mount)的区别?_**

```
* 绑定挂载用于将宿主机上任意的文件夹挂载到容器中的文件系统中

* 卷本质上是将由宿主机上Docker服务器负责管理的文件夹挂载到容器的文件系统中
```

**_如何使用 Docker Compose 启动多容器应用?_**

```
在compose.yaml文件中配置多个容器的镜像和参数, 使用docker compose up/down 启动或关闭一个多容器应用
```

**_`.dockerignore`文件的作用?_**

```
指明在镜像构建时不需要COPY的文件
```

**_讲一下镜像为什么分层?_**

```
当镜像rebuild时, 采用分层可以只重新构建更改了的镜像层, 提高构建速度
```
