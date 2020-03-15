---
layout: post
title:  "[初级]-Docker-学习笔记"
date:   2020-03-14 21:18:54
categories: Docker
tags: Docker Notes
excerpt: Docker学习笔记，上传本地笔记。
mathjax: true
---

* content
{:toc}

> Docker是一个基于基于Golang的开源项目，Docker 项目的目标是实现**轻量级的操作系统虚拟化**解决方案，由于工作需要作者开始学习Docker。

## Docker简介

* Docker容器本质上是宿主机上的一个进程， 通过namespace实现了资源隔离，通过 cgroups 实现了资源的限制，通过写时复制机制（copy-on-write）实现了高效的文件操作。

* Docker有五个命名空间：进程、网络、挂载、宿主和共享内存，为了隔离有问题的应用，Docker运用Namespace将进程隔离，为进程或进程组创建已隔离的运行空间，为进程提供不同的命名空间视图。这样，每一个隔离出来的进程组，对外就表现为一个container(容器)。

## 容器的基本概念

* 镜像：是一种*轻量级*、*可执行*的独立软件包，他包含了运行某个软件所需的*所有内容*，包括代码、运行时、库、环境变量和配置文件。

* 容器：是镜像的*运行时实例*，实际执行时镜像会在内存中。默认情况下，它作为守护线程独立于主机环境运行，仅在配置为访问主机文件和端口的情况下才会执行此操作。

## 实际操作

#### **ubuntu安装docker**

1.``` ~$ wget -qO- https://get.docker.com/ | sh```

2.以非root用户可以直接运行docker时，需要执行 sudo usermod -aG docker runoob 命令，然后重新登陆

3.```docker version``` 检查版本并且是否安装成功

4.启动docker后台服务```sudo service docker start```

5.检查是否安装成功，运行hello-world ```docker run hello-world```

#### **在docker运行一个容器并且执行hello world的应用程序**

```
docker run ubuntu:15.10 /bin/echo "Hello world"
```

-docker: Docker 的二进制执行文件

-run: 与docker组合来运行一个容器

-ubuntu:15.10: 指定要运行的镜像和镜像版本，docker首先在本地主机上找镜像，如果存在，创建新容器。不存在docker会从镜像残酷docker hub上下载公共镜像

-/bin/echo "Hello world!" 在启动的容器里面执行的命令

#### **运行交互式的容器**

```
docker run -i -t ubuntu:15.10 /bin/bash
```

-t: 在新容器中指定一个伪终端或者终端

-i: 允许你对容器内的标准输入（STDIN）进行交互，如果没有该参数，将无法进行交互

#### **以后台模式（线程方式）启动容器**

```
docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done
```

-d: 以线程方式运行一个容器的参数

#### **运行一个web运用**
将在docker容器中运行一个 Python Flask 应用来运行一个web应用

1.```docker pull training/webapp``` # 载入镜像

2.```docker run -d -P training/webapp python app.py```

-d: 让容器在后台运行

-P: 让容器内部使用的网络端口映射到我们使用的主机上
        
3.可以使用-p参数来设置不一样的端口 
```
docker run -d -p 5000:5000 training/webapp python app.py
```
容器内部的5000端口映射到我们本地主机的5000 端口上

#### **查看刚刚运行的web应用容器**

```
docker ps
``` 
PPRTS 0.0.0.0：32769->5000/tcp, 表示docker内部的5000的端口映射到主机的32769端口

#### **查看WEB应用程序容器内部的进程**

```
docker top [container id|name]
```

#### **检查 WEB 应用程序**

```
docker inspect
``` 
来查看 Docker 的底层信息。它会返回一个 JSON 文件记录着 Docker 容器的配置和状态信息

#### **列出本地主机镜像**

```
docker images
 ```
![](\images\docker-learning-notes\docker-images.png)

-repository: 镜像仓库源

-Tag: 镜像的版本或者标签

-Image ID: 镜像ID

-Created: 镜像创建的时间

-Size: 镜像大小

同一仓库源可以有多个Tag，代表这个仓库源的不同个版本，如ubuntu仓库源里，有15.10、14.04等多个不同的版本，我们使用 REPOSITORY:TAG 来定义不同的镜像
```
docker run ubuntu:15.10 /bin/echo "Hello world"
```
如果你不指定一个镜像的版本标签，例如你只使用ubuntu，docker将默认使用 ubuntu:latest镜像

#### **获取一个新的镜像**

```
docker pull repository:tag
```
默认从docker hub上下载

#### **查找镜像**

查看在docker hub上是否存在某个镜像，使用：

```
-docker search [repository]
```

-NAME: 镜像仓库源名称

-DESCRIPTION：镜像描述

-OFFICIAL： 是否dicker 官方发布

#### **创建镜像**

当我们从docker镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改

-从已经创建的容器中更新镜像，并且提交这个镜像

-使用dockerfile 指令来创建一个新的镜像

* 更新镜像

    1.更新镜像前先创建一个容器
    ```
    docker run -t -i ubuntu:15.10 /bin/bash
    ```

    2.在运行的容器内使用 ```apt-get update``` 命令进行更新

    3.使用```exit```来退出这个容器

    4.使用commit提交更改后的容器副本到本地主机镜像仓库

    ```
    docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2
    ```
    -m: 提交的描述信息

	-a: 指定镜像作者

	-e218edb10161: 容器ID

    -runoob/ubuntu:v2:指定要创建的目标镜像名和tag

* 创建镜像

    1.创建一个dockerfile的文件 包含一組指令告訴docker如何構建我們的鏡像

    ```
    FROM    centos:6.7
	MAINTAINER      Fisher "fisher@sudops.com"
	
	##镜像内执行命令
	RUN     /bin/echo 'root:123456' |chpasswd
	RUN     useradd runoob
	RUN     /bin/echo 'runoob:123456' |chpasswd
	RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
	EXPOSE  22 ##暴露端口
	EXPOSE  80
	CMD     /usr/sbin/sshd -D

    ```

    每一个指令都会在镜像上创建一个新的层，每一个指令的前缀都必须是大写的.

    2.我们使用 Dockerfile 文件，通过 ```docker build``` 命令来构建一个镜像 

    ```
    docker build -t runoob/centos:6.7 .
    ```

    -t: 指定要创建的目标镜像名

    -**.**: 指定DockerFile文件的目录，可以是绝对路径也可以是相对

    3.使用 ```docker images``` 查看创建的镜像

    4.使用run來根据新的镜像创建容器

#### **设置镜像标签**

```
docker tag [images id] [username/repository name:tag]
```

#### **查看容器映射的主机端口**

运行命令```docker port $container_id | $container_name```可以查看docker映射的主机端口号，除了端口号，该命令只能显示容器运行时端口绑定信息。


## 定义网络规则来设置容器间的通信

1.通过 ```expose``` 参数在运行时暴露端口

2.在DockerFile中使用```EXPOSE```命令

3.在Docker run的時候通過-p或者-P參數來發布端口。**靠谱**

4.通过--link链接容器 **慎用**

* 区别：

    -1和2作用相同，使容器向主机暴露一个或者一个范围的端口，但并不绑定
    
    -```--expose```设置可接受的端口范围 ```eg:--expose=2000-3000```

    -使用-p参数显示將一个或者一组端口从容器裡绑定到宿主机器上，而不仅仅是提供一个端口。

    -p参数有几种不同的格式：

    ```
    ip:hostPort:containerPort
    ip::containerPort 
    hostPort:containerPort
    containerPort
    ```

    实际中，可以忽略ip或者hostPort，但是必须要指定需要暴露的containerPort。另外，所有这些发布的规则都默认为tcp。如果需要udp，需要在最后加以指定，比如```-p  1234:1234/udp```

* 避免冲突的最佳方法是让Docker自己分配hostPort。可以使用```docker run -p 3000 my_image```来运行容器，而不是显式指定宿主机端口。这时，Docker会帮助选择一个宿主机端口。

* Expose -P一起使用: 因为EXPOSE通常只是作为记录机制，也就是告诉用户哪些端口会提供服务，Docker可以很容易地把Dockerfile里的EXPOSE指令转换成特定的端口绑定规则。只需要在运行时加上-P参数，Docker会自动为用户创建端口映射规则，并且帮助避免端口映射的冲突。

* 总结

    ![](\images\docker-learning-notes\port-summary.png)

## Docker 命令

[More Information](https://010sec.cn/2017/09/20/install-docker-and-using.html)

```
docker ps -- print all information of running dockers 
    container id:container id, is unique for each container.
    names:  container name that allocates automatically.
	
docker logs [-f] [container id | names]-- print the logs of the container. 
    -f: 让 docker logs 像使用 tail -f 一样来输出容器内部的标准输出

docker stop  [container id | names]-- stop a running container.
    命令会向运行中的容器发送一个SIGTERM的信号，然后停止所有的进程
	
docker pull [image] 从docker hub中載入鏡像

docker port [container id | names] 可以查看指定 （ID 或者名字）容器的某个确定端口映射到宿主机的端口号

docker inspect [container id | names] 可以查看docker容器配置和状态信息。

docker start  [container id | names] 重启容器 ,为容器文件系统创建了一个进程隔离空间。注意，每一个容器只能够有一个进程隔离空间。

docker restart [container id | names]正在运行的容器，我们可以使用该命令来重启

docker ps 
    -l: 查看最后一次创建的容器 
    -a: 列出所有容器，不管是運行態還是非運行態

docker ps 命令会列出所有运行中的容器。这隐藏了非运行态容器的存在，如果想要找出这些容器，我们需要使用下面这个命令。

docker rm [container id | names] 移除容器 需要在停止的状态删除

docker images 列出本地主机上的镜像,docker images命令会列出了所有顶层（top-level）镜像。实际上，在这里我们没有办法区分一个镜像和一个只读层，所以我们提出了top-level镜像。只有创建容器时使用的镜像或者是直接pull下来的镜像能被称为顶层（top-level）镜像，并且每一个顶层镜像下面都隐藏了多个镜像层
    –a: 列出所有的可读层

docker run -i -t ubuntu:15.10 /bin/bash 运行交互式的容器 ubuntu:15.10指定镜像仓库源和版本 docker create + docker start

docker run <image>:<tag> 指定版本号的镜像的运行

docker run -p [hostip]:[dockerip] -d [imagename] 以守护线程模式起一个容器，并为他指定暴露的端口和映射的主机端口

docker search [repository] 查看在docker hub中是否存在某个镜像仓库

docker commit 提交一个容器副本到本地主机镜像仓库中

docker tag [images id] [username/repository name:tag] eg: docker tag 860c279d2fec runoob/cenos:dev  設置鏡像標籤 !!!一個鏡像可以有多個標籤

docker create  命令为指定的镜像（image）添加了一个可读可写层，构成了一个新的容器。注意，这个容器并没有运行

docker history 查看某一个image-id下的所有层

docker kill  向所有运行在容器中的进程发送了一个不友好的SIGKILL信号

docker ps -a | grep [search string] 搜索指定名字的容器

docker exec -it [docker-name] bash 在docker-name中运行bash

docker push [image] 将image推到镜像仓库

```

## Docker理解

* Namespace

    命名空间 (namespaces) 是 Linux 为我们提供的用于分离进程树、网络接口、挂载点以及进程间通信等资源的方法。

    Linux 的命名空间机制提供了以下七种不同的命名空间，包括 CLONE_NEWCGROUP、CLONE_NEWIPC、CLONE_NEWNET、CLONE_NEWNS（隔离挂载点）、CLONE_NEWPID、CLONE_NEWUSER 和 CLONE_NEWUTS，通过这七个选项我们能在创建新的进程时设置新进程应该在哪些资源上与宿主机器进行隔离。

    Docker 通过 Linux 的命名空间实现了网络的隔离，又通过 iptables 进行数据包转发，让 Docker 容器能够优雅地为宿主机器或者其他容器提供服务。

* Cgroups 

    物理资源隔离，例如：CPU、内存、磁盘 I/O 和网络带宽；使用文件系统來实现。

    每一个CGroup都是一组被相同的标准和参数限制的进程，不同的 CGroup 之间是有层级关系的，也就是说它们之间可以从父类继承一些用于限制资源使用的标准和参数。

    不同的CGroup之间是有层级关系的，也就是说它们之间可以从父类继承一些用于限制资源使用的标准和参数。

* 存储驱动

    Docker 中的每一个镜像都是由一系列只读的层组成的，Dockerfile 中的每一个命令都会在已有的只读层上创建一个新的层。

## 运行容器过程解释

当我们想运行一个容器的时候，docker会：
	
* 拉取镜像，若本地已经存在该镜像，则不用到网上去拉取
* 创建新的容器
* 分配union文件系统并且挂着一个可读写的层，任何修改容器的操作都会被记录在这个读写层上，你可以保存这些修改成新的镜像，也可以选择不保存，那么下次运行改镜像的时候所有修改操作都会被消除
* 分配网络\桥接接口，创建一个允许容器与本地主机通信的网络接口
* 设置ip地址，从池中寻找一个可用的ip地址附加到容器上，换句话说，localhost并不能访问到容器
* 运行你指定的程序
* 捕获并且提供应用输出，包括输入、输出、报错信息

## 参考:

[Docker 教程](https://www.runoob.com/docker/docker-tutorial.html)

[docker安装与使用](https://010sec.cn/2017/09/20/install-docker-and-using.html)

[Docker 中文文档](https://doc.yonyoucloud.com/doc/chinese_docker/ABOUTDOCKER.html)

[Docker 核心技术与实现原理](https://draveness.me/docker)

[Docker网络原则入门：EXPOSE，-p，-P，-link](http://dockone.io/article/455)




































