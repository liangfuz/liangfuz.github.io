---
layout: post
title: 2.docker registry私有仓库搭建
category: docker
tags: docker
keywords: docker
---


## 1. 预备条件
   <a href = "https://docs.docker.com/engine/installation/">Docker版本1.16或以上</a>
   
   快速测试<a href = "https://blog.developabc.com/2019/04/01/docker-install.html">环境</a>是否安装正确：
   ```
   docker run hello-world
   ```
## 2. 介绍

   docker仓库是一个高度可伸缩,开发者能够存储和发布镜像的应用，它是遵循<a href="http://en.wikipedia.org/wiki/Apache_License">Apache license</a>,
   开源的项目
   
    ```
    Stack
    Services
    Container
    ```
   一般情况下，如果要开始编写Python应用程序，首先要做的就是在机器上安装Python运行环境。
   这就面临一种情况，我们的机器上面的环境要和应用完美契合，同时也要在运行的生产环境契合。
   而使用Docker，只需要一个基本的可运行的Python镜像，然后构以此为基础，便能确保应用程序、和它的依赖项一起运行。

## 3 docker registry安装及使用
   
   
### 3.1 启动registry：
   ```
   [root@master build]# docker run -d -p 5000:5000 --name registry registry:2
   Unable to find image 'registry:2' locally
   2: Pulling from library/registry
   c87736221ed0: Pull complete 
   1cc8e0bb44df: Pull complete 
   54d33bcb37f5: Pull complete 
   e8afc091c171: Pull complete 
   b4541f6d3db6: Pull complete 
   Digest: sha256:77a8fb00c00b99568772a70f0863f6192ff2635e4af4e22e4d9c622edeb5f2de
   Status: Downloaded newer image for registry:2
   c095faf0055772bc0d5e7e7e96f68156f51a1e995fc9f08d4d10bbf2681bce8c
   ```
### 3.2 从docker hub随便下载一个镜像(也可以自己build一个)
   ```
   [root@master build]# docker pull ubuntu
   Using default tag: latest
   latest: Pulling from library/ubuntu
   6abc03819f3e: Pull complete 
   05731e63f211: Pull complete 
   0bd67c50d6be: Pull complete 
   Digest: sha256:f08638ec7ddc90065187e7eabdfac3c96e5ff0f6b2f1762cf31a4f49b53000a5
   Status: Downloaded newer image for ubuntu:latest
   ```
### 3.3 给镜像打上标签，并指向我们启动的仓库
   ```
   [root@master build]# docker image tag ubuntu localhost:5000/myfirstimage
   ```
### 3.4 将镜像上传到我们的仓库
   ```
   [root@master build]# docker push localhost:5000/myfirstimage
   The push refers to repository [localhost:5000/myfirstimage]
   8d267010480f: Pushed 
   270f934787ed: Pushed 
   02571d034293: Pushed 
   latest: digest: sha256:b36667c98cf8f68d4b7f1fb8e01f742c2ed26b5f0c965a788e98dfe589a4b3e4 size: 943
   ```
### 3.5 从仓库下载镜像
   ```
   [root@master build]# docker pull localhost:5000/myfirstimage
   Using default tag: latest
   latest: Pulling from myfirstimage
   Digest: sha256:b36667c98cf8f68d4b7f1fb8e01f742c2ed26b5f0c965a788e98dfe589a4b3e4
   Status: Image is up to date for localhost:5000/myfirstimage:latest
   ```
   如果下载镜像的时候报如下错误
   ```
   [root@worker1 ~]# docker pull 192.168.10.152:5000/fdlyhello
   Using default tag: latest
   Error response from daemon: Get https://192.168.10.152:5000/v2/: http: server gave HTTP response to HTTPS client
   ```
   这个问题可能是由于客户端采用https，docker registry未采用https服务所致
   解决办法是修改`/etc/docker/daemon.js`文件，在文件中写入：
   ```
   { "insecure-registries":["192.168.10.152:5000"] }
   ```
   保存退出后，重启docker。问题解决：
   ```
   [root@worker1 ~]# docker pull 192.168.10.152:5000/fdlyhello
   Using default tag: latest
   latest: Pulling from fdlyhello
   27833a3ba0a5: Already exists 
   8b35abcb27de: Already exists 
   cd1fc6dee9fe: Already exists 
   2c6a92003566: Already exists 
   1fd3a406cb45: Pull complete 
   86258feb7a97: Pull complete 
   2bdc9d83a3a1: Pull complete 
   Digest: sha256:78a6b7802c3a3a24bc8deacdbf01796cca029ff24ef18392a3f003ad42b0adab
   Status: Downloaded newer image for 192.168.10.152:5000/fdlyhello:latest
   ```
### 3.6 停止仓库并删除所有数据
   ```
   docker container stop registry && docker container rm -v registry
   ```
   