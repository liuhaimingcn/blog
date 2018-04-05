title: Docker Hello World
date: 2016-10-9  
tags:
    - original
categories:
    - Docker
---

## 查看linux版本

```sh
$ lsb_release -a
LSB Version:    :core-4.1-amd64:core-4.1-noarch
Distributor ID: CentOS
Description:    CentOS Linux release 7.2.1511 (Core)
Release:    7.2.1511
Codename:   Core
```

<!-- more -->   

## Docker 环境准备

* CentOS7 系统 CentOS-Extras 库中已带 Docker，可以直接安装

### 安装

```sh
$ su root
# yum install docker
```

### 启动

```sh
# service docker start
Redirecting to /bin/systemctl start  docker.service
```

### 让它随系统启动自动加载

```sh
# service docker start
Redirecting to /bin/systemctl start  docker.service
# chkconfig docker on
注意：正在将请求转发到“systemctl enable docker.service”。
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```

## 镜像

### 获取镜像
```sh
# docker pull ubuntu:12.04
Trying to pull repository docker.io/library/ubuntu ...
12.04: Pulling from docker.io/library/ubuntu
36cef014d5d4: Pull complete
0d99ad4de1d2: Pull complete
3e32dbf1ab94: Pull complete
44710c456ffc: Pull complete
56e70ac3b314: Pull complete
Digest: sha256:0c25aa67baaff2b895882ce1e7d25efeeb15d0f38df6c099e23f481641cd6cab
Status: Downloaded newer image for docker.io/ubuntu:12.04
```

### 列出本地已有的镜像

```sh
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/ubuntu    12.04               e216a057b1cb        12 days ago         103.6 MB
```
* REPOSITORY: 来自于哪个仓库
* TAG: 镜像的标记（如果不指定具体的标记，则默认使用 latest 标记信息。）
* IMAGE ID: 它的 ID 号（唯一）
* CREATED: 创建时间
* SIZE: 镜像大小

### 创建镜像

* 修改已有镜像

```sh
# docker run -t -i docker.io/ubuntu:12.04 /bin/bash
root@d7ac9b2f8cd7:/# apt-get update
root@d7ac9b2f8cd7:/# apt-get install nodejs
root@d7ac9b2f8cd7:/# node -v
v0.6.12
root@d7ac9b2f8cd7:/# exit
exit
# docker commit -m "add node evn" -a "haiyue" d7ac9b2f8cd7 haiyue/nodejs:v1
sha256:23c1c51f86414ae2ab3bf31f192537e6635c248044b61802c9e553ef4e46fbf9
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
haiyue/nodejs       v1                  23c1c51f8641        10 seconds ago      162.4 MB
docker.io/ubuntu    12.04               e216a057b1cb        12 days ago         103.6 MB
```

-m 来指定提交的说明信息；-a 可以指定更新的用户信息；之后是用来创建镜像的容器的 ID；最后指定目标镜像的仓库名和 tag 信息。创建成功后会返回这个镜像的 ID 信息。

* 利用 Dockerfile 来创建镜像
```sh
# mkdir haiyue
# cd haiyue/
# This is a comment
# vim Dockerfile
FROM docker.io/ubuntu:12.04
MAINTAINER haiyue
RUN apt-get update
RUN apt-get install nodejs
# docker build -t="haiyue/nodejs:v2" .
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
haiyue/nodejs       v2                  5190c35fc391        4 seconds ago       103.6 MB
haiyue/nodejs       v1                  23c1c51f8641        2 hours ago         162.4 MB
```

```
Dockerfile 基本的语法是
#: 注释
FROM: 告诉 Docker 使用哪个镜像作为基础
MAINTAINER: 维护者的信息
RUN: 指令会在创建中运行

build 指令后 -t 标记来添加 tag，指定新的镜像的用户信息。 “.” 是 Dockerfile 所在的路径（当前目录），也可以替换为一个具体的 Dockerfile 的路径。

*注意一个镜像不能超过 127 层
```

* 上传镜像(推送自己的镜像到仓库, Docker Hub)
```sh
# docker push haiyue/nodejs
The push refers to a repository [haiyue/nodejs] (len: 1)
Sending image list
Pushing repository haiyue/nodejs (2 tags)
```

### 存出和载入镜像

* 存出镜像

```sh
# docker save -o nodejs_v1.tar haiyue/nodejs:v1
```

* 载入镜像

```sh
# docker load < nodejs_v1.tar
```

### 移除

```sh
# docker rmi haiyue/nodejs:v2
Untagged: haiyue/nodejs:v2
Deleted: sha256:5190c35fc391b6e9b3ed228a60a8342887fb555efcbb8a7bc351badfbb306550
Deleted: sha256:01e2cd409ed3b15a1c34a2cd656772f51e472ec988b2011ec7d60de3a23e3111
Deleted: sha256:daf78048142d52646c96e8ac025ea5d034270e15cab677838638767b00cb908e
# docker rmi 23c1c51f8641
Failed to remove image (23c1c51f8641): Error response from daemon: conflict: unable to delete 23c1c51f8641 (must be forced) - image is being used by stopped container 1586df1c5dc1
# docker rmi -f 23c1c51f8641
Untagged: haiyue/nodejs:v1
```

## 容器

### 启动

* 新建并启动
```sh
# docker run haiyue/nodejs:v1 /bin/echo 'Hello world'
Hello world
# docker run -t -i haiyue/nodejs:v1 /bin/bash
root@66230fd659ff:/# node -v
v0.6.12
root@66230fd659ff:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  selinux  srv  sys  tmp  usr  var
```

```
-t 让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上 
-i 则让容器的标准输入保持打开。

docker run 来创建容器时，Docker 在后台运行的标准操作包括：
检查本地是否存在指定的镜像，不存在就从公有仓库下载
利用镜像创建并启动一个容器
分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
从地址池配置一个 ip 地址给容器
执行用户指定的应用程序
执行完毕后容器被终止
```

* 启动已终止容器
```
利用 docker start 命令，直接将一个已经终止的容器启动运行。
docker restart 命令会将一个运行态的容器终止，然后再重新启动它。
```

### 守护态运行

```sh
# docker run -d haiyue/nodejs:v1 /bin/sh -c "while true; do echo hello world; sleep 1; done"
90a4f5809293e21f8eacf1945668b9076e68ed500ac3d3018fc902d947f57248
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
90a4f5809293        haiyue/nodejs:v1    "/bin/sh -c 'while tr"   9 seconds ago       Up 8 seconds                            determined_sammet
# docker logs 90a4f5809293
hello world
hello world
hello world
```

### 终止容器

```sh
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
90a4f5809293        haiyue/nodejs:v1    "/bin/sh -c 'while tr"   2 minutes ago       Up 2 minutes                            determined_sammet
# docker stop 90a4f5809293
90a4f5809293
# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                        PORTS               NAMES
90a4f5809293        haiyue/nodejs:v1    "/bin/sh -c 'while tr"   3 minutes ago       Exited (137) 37 seconds ago                       determined_sammet
```

### 进入容器

```sh
# docker attach 90a4f5809293
hello world
hello world
hello world
```

### 导出和导入容器

* 导出容器

```sh
# docker export 90a4f5809293 > determined_sammet.tar
# ls
determined_sammet.tar
```

* 导入容器快照(容器快照文件中再导入为镜像)

```sh
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
haiyue/nodejs       v1                  23c1c51f8641        3 hours ago         162.4 MB
docker.io/ubuntu    12.04               e216a057b1cb        12 days ago         103.6 MB
# cat determined_sammet.tar | docker import - haiyue/nodejs:v2
sha256:7b78999d774ccb41d474fb2c851323191967593053a26982c11157c7621e9202
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
haiyue/nodejs       v2                  7b78999d774c        3 seconds ago       142.1 MB
haiyue/nodejs       v1                  23c1c51f8641        3 hours ago         162.4 MB
docker.io/ubuntu    12.04               e216a057b1cb        12 days ago         103.6 MB
```

```
用户既可以使用 docker load 来导入镜像存储文件到本地镜像库，也可以使用 docker import 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。
```

* 删除

```
可以使用 docker rm 来删除一个处于终止状态的容器。
如果要删除一个运行中的容器，可以添加 -f 参数。Docker 会发送 SIGKILL 信号给容器。
```

<br>