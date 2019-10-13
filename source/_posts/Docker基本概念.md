title: Docker基本概念
author: Answer
tags:
  - Docker
categories:
  - Docker
date: 2019-10-13 14:58:00
---
# 基本概念

- 镜像（Image）：Docker 镜像是特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

- 分层存储：镜像由一组文件系统组成，或者说由多层文件系统联合组成，镜像构建会一层层构建，前一层是后一层的基础。

- 容器（Container）：容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

- 容器存储层：为容器读写而准备的存储层，生命周期与容器一样。

- 数据卷：数据卷直接对宿主主机进行读写，生存周期独立于容器，容器消亡，数据卷不会消亡。

- 仓库（Repository）：存放镜像的地方，供其它服务器使用，类似 Maven 仓库

- 虚悬镜像：拉取镜像时候，新镜像和旧镜像同名，原来镜像的仓库门、标签名就变为 none

- 中间层镜像：镜像构建是一层层的，相同的镜像之会构建一次，然后复用。慎重删除，因为其他镜像可能依赖于它。

# docker 命令

docker pull 拉取镜像

docker run -it ubuntu:18.04 bash  运行镜像并拉起 Bash 命令行

exit 退出容器

docker image ls 列出已下载的顶层镜像，Size 表示各层所占空间总和

- Docker Hub 大小是压缩过的，本地是解压后占用的，所以大小不一致

- 分层构建，多个镜像如果使用相同的层，那么只会占用一份存储空间

docker system df 查看镜像、容器、数据卷占用空间

docker image prune 删除虚悬镜像

docker image ls -a 列出所有镜像，包括中间层镜像

docker image ls ubuntu 根据仓库名列出镜像

docker image ls ubuntu:18:04

docker image ls -f since=mongo:3.2 列出 mongto:3.2 之后的镜像， -f 是 filter 缩写， since 也可以换成 before

```
docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}" 模板方法，只列出部分，标题也自定义
```

- 镜像删除

docker image rm 删除镜像，可以根据 imageID、镜像名、摘要（sha字段）
    docker image rm 501 根据 ImageID 前 3 位就可以区分删除
    docker image rm centos，根据镜像名
docker image rm $(docker image ls -q redis) 删除仓库名为 redis 的镜像
    
删除分为两步：Untagged 和 Deleted，一个镜像可能多个标签，所以先 untag 某个标签的镜像；如果一个镜像没有任何依赖，那么就触发 deleted。

# Dockerfile 制作镜像
```
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
RUN apt-get update
```
FROM 指定基础镜像
FROM scratch：这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。接下来所写的指令将作为镜像第一层开始存在。
RUN 执行命令，Dockerfile 中每一个指令都会建立一层，然后 commit。要注意镜像是一层层构建的，所以要注意构建的方式。

docker build - nginx:v3 . ：构建镜像，最后的 “.” 表示当前目录（上下文路径）

镜像构建上下文：镜像构建是在 Docker 引擎完成，本地文件会打包上传过去。

docker build - < Dockerfile ：从 Dockerfile 构建镜像

- COPY：复制文件
- CMD：启动容器，指定所运行的程序及参数，CMD [ "sh", "-c", "echo $HOME" ]
- ENTRYPOINT：入口点，
- ENV：设置环境变量，ENV NODE_VERSION 7.2.0，$NODE_VERSION 使用
- EXPOSE：声明容器打算使用什么端口，而 -p <宿主端口>:<容器端口> 是映射宿主端口和容器端口
- WORKDIR：指定工作目录，以后各层的构建以此为基础
- 


# 容器

docker run -t -i ubuntu:18.04 /bin/bash 启动一个容器并打开终端  

docker container start 启动一个已终止的容器

docker container logs 获取容器输出信息

docker container stop 终止容器

docker exec -it 69d1 bash 进入容器，退出不会终止容器


# 博客参考

[Docker — 从入门到实践](https://github.com/yeasy/docker_practice)