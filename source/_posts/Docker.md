---
title: Docker简述
date: 2024-07-21 23:58:31
category: Cloud Native
tags:
  - Docker 
  - Kubernetes
---

<h2>Docker的基本概念</h2>


<h3>解决的问题<h3/>
1. 统一标准

    应用构建将统一通过docker build来打包成镜像文件，无论是什么语言，java python C++ 都可以通过Docker构建出同样的软件包


2. 应用分享

    所有的docker景象都会被放在统一的地方 Docker hub。 这就类似于安卓的应用商城，所有的镜像文件都可以在一个地方下载


3. 应用运行

    因为我们拥有了统一的镜像文件，那么不过是什么语言构建的应用，都可以通过 Docker run命令来运行，解决了不同语言需要通过不同步骤运行的问题




<h2>Docker的容器化</h2>



<h3>虚拟化技术<h3/>
我们可以通过安装虚拟机的方式，将一台高配置的主机分割成多台小主机，这样可以将主机的性能最大化，并且防止一些小程序崩溃或者内存泄漏影响所有应用


但是同样虚拟机也会有一些问题，如下：


<br>
<img src="/blog.io/img/虚拟化技术.png">
<br>


为了更轻量级的使用，doker提供了更为轻量的服务，首先Doker提供了公用的OS，每个镜像里只需要带上自己额外需要的依赖。同时如果需要移植，只需要把镜像文件移植到新的Docker机器里就可以了

<br>
<img src="/blog.io/img/容器化技术.png">
<br>




<h2>Docker的架构</h2>

Docker的架构十分简单，首先你需要在你本地客户端使用dicker命令来进行操作。其次在你的远程主机上你需要下载一个Docker并且运行他的后台进程Docker Daemon。
然后你就可以在Docker Hub里找到你想要使用的软件，并且将它下载到你的远程主机 Docker Host上。这时你的Docker Host上就会拥有了各种镜像，使用命令行将这些镜像
运行在每一个不同的容器Container里就可以了


<br>
<img src="/blog.io/img/Docker架构.png">
<br>


<h2>Docker中的命令</h2>


<br>
<img src="/blog.io/img/Docker命令.png">
<br>


1. 下载docker镜像

   Docker pull nginx
   (latest version)

   Docker pull nginx:1.20.1

2. 查看本地中的镜像

   docker images

3. 删除镜像

   docker rmi redis:6.2.4

4. 启动容器
   
   docker run 设置项 镜像名 镜像启动的命令（Args）

   docker run --name=myngix -d nginx:latest

   (-d是后台运行)
   (注意已经使用过的名字不能再次使用，哪怕之前的容器已经停止运行了（docker stop id），还是需要运行docker rm id 来删除掉之前的容器)

   再次启动 docker start id

5. 查看那些程序在运行

   docker ps 查看在运行的
   docker ps -a查看所有

6. 更新启动项
   
   docker update id/名字 --restart=always

   将容器的启动项更新成每次docker启动容器也会自动重启

7. 将容器端口和主机端口进行映射
   
   docker run --name=myngix -d --restart=always -p 88:80 nginx

   这里 -p就是端口映射， 主机端口：容器的端口

   





