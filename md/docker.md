---
title: "Docker初步学习"
date: 2019-09-16T21:28:27+08:00
lastmod: 2019-09-25T11:54:20+08:00
draft: false
tags: ["Docker", "container"]
categories: ["code"]
author: "Rouzip"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

> Docker 是今年来流行的 container 工具，其为开发与运维人员提供了方便的环境，简化了开发与部署的流程。这篇文章将会将其与虚拟机进行比较，并在最后介绍一些 Docker 的常用操作。

## Docker 创造背景

Docker 的理论基础容器(container)，早在它开发出之前就被人提出了。面对着越加复杂的开发与部署环境，开发人员通常会花费很长的时间来进行环境的搭建，更有甚者花费一天的时间只为了打出一句`hello world` 。所以为了更好地将数据进行分发和解决依赖问题，进而在各个机器上直接运行。基于 Linux 上的`namespace` 和 `cgroup` 作为基础，Docker 的原型被开发出来，其中`namespace` 就可以做到`container` 的作用，而`cgroup` 则是控制系统的资源。

## 理论基础

Docker 中最重要的就是三个概念：镜像，镜像仓库和容器，这三个概念组成了整个 Docker 架构。

### 镜像

镜像(image)是自己程序与环境的打包集合，最初的镜像就是一个简单的联合文件系统加上必要的 runtime 和配置等，每次对这个镜像的修改则是在最初的镜像上添加新的一层作为修改。一开始我将其与虚拟机的镜像所混淆，其实这两者的概念完全不同。虚拟机的镜像更像是光盘记录，是只读的，所运行的只是镜像中的安装程序；而 Docker 中的镜像是一次次构建后的产物，上一次删除后的文件仍旧存在，其只是被当前操作的这一层所覆盖。因此每次的构建都要十分谨慎，否则会产生很多无用的改动。

### 容器

拿面向对象中的概念来举例，容器就是 OOP 中的实例，而镜像则像是类中的`constructor`，是抽象的一层概念，具体的使用还是要进行“实例化”。但是容器还是与直接运行的程序所不同，其是在 Docker 引擎上所执行的程序，具有独立的`namespace`，即与其他的程序所隔离，pid 独立，资源独立等，但是内核仍旧和宿主机相同。其存储层信息也是随着 container 的删除而消失。同时根据 Docker 的最佳实践原则，其不应该在存储层直接进行读写，而是和绑定的 Volume 进行读写，或者和宿主机的特定目录进行绑定，这样就保证了数据的安全性。

### 仓库

仓库的概念和我们所熟知的 Git 类似，大家可以将自己定制好的镜像推送到中心服务器，在需要的时候拉取下来。同时，Docker 也具有私有仓库和共有仓库的概念，最大的共有仓库 Docker Hub 就类似于 Github，用户们把镜像上传到这上边，然后可以根据标签拉取合适的镜像来使用。

## 常用 Docker 命令

### image 相关命令

```bash
$ docker run -it ubuntu bash
# i表示可以交互 t表示开启终端交互，在本地没有镜像时会从Docker Hub拉取
$ docker image ls (-a)
# 显示本地的所有镜像文件
$ docker inspect images
# 查看镜像详细信息
$ docker image rm [imageName]
# 删除某image文件
$ docker search [imageName]
# 搜索某image文件
$ docker image pull/push [imageName]
# 拉取或者推送某镜像
```

### contrainer 相关命令

```bash
$ docker container run [imageName]
# 在容器中运行镜像文件
$ docker container ls --all
# 列出本机正在运行的所有的容器(包含退出的容器)
$ docker start [ContainerID]
# 启动某一个镜像
$ docker top [ContainerID]
# 查看容器进程
$ docker logs [ContainerID]
# 查看容器日志
$ docker diff [ContainerID]
# 查看容器变化
$ docker ps
# 查看容器
$ docker port [ContainerID]
# 查看容器端口
$ docker inspect [ContainerID]
# 查看容器详情
```

## 参考文献

1. <https://juejin.im/post/5b260ec26fb9a00e8e4b031a>
