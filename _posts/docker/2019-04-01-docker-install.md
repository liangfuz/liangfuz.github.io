---
layout: post
title: centos安装docker
category: docker
tags: docker
keywords: docker
---

## 1. docker介绍
Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化，容器是完全使用沙箱机制，相互之间不会有任何接口。

Docker从1.13版本之后采用时间线的方式作为版本号，分为社区版CE和企业版EE。
社区版是免费提供给个人开发者和小型团体使用的，企业版会提供额外的收费服务，比如经过官方测试认证过的基础设施、容器、插件等。


## 2. docker安装

### 2.1 系统要求
安装docker ce需要centos7版本
必须启用centos-extras仓库。默认情况下是自动启动的，但如果您禁用了它，则需要重新<a href="https://wiki.centos.org/AdditionalResources/Repositories">启用</a>它。

### 2.2 卸载老版本的docker
老版本的docker名称是docker或者docker-engine，如果安装了的话需要卸载老版本以及所依赖项
```
yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate 
docker-logrotate docker-engine
```
如果yum报告说没有安装这些包，也是OK的。

### 2.3 安装docker-ce
根据个人需求，docker安装可以分为以下几种方式：

  1）大多数用户会使用<a href="https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-repository">docker仓库</a>的方式安装，这种便于安装和更新，推荐使用
  
  2）部分用户会自行下载RPM包来<a href="https://docs.docker.com/install/linux/docker-ce/centos/#install-from-a-package">手动安装</a>和手动升级，这种适用于无网络访问的系统
  
  3）在测试和开发环境中，一些用户会选择使用自动化的<a href="https://docs.docker.com/install/linux/docker-ce/centos/#install-using-the-convenience-script">方便的脚本</a>来安装Docker。
  
  PS:本实例只涉及第一种方式
### 2.4 设置仓库
  
  第一次安装docker需要设置docker仓库，之后就可以从仓库选择安装或者更新

  1）安装所需要的库，yum-utils提供了yum-config-manager工具，devicemapper依赖device-mapper-persistent-data和lvm2
  
  ```
  yum install -y yum-utils device-mapper-persistent-data lvm2
  ```
  
  2）设置稳定的docker仓库
  
  ```
  yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  ```
  
### 2.5 安装docker-ce
  
  1）安装最新版：
  
  ```
  yum install docker-ce docker-ce-cli containerd.io
  ```
    
  2）安装指定版本：
  a.列出可用版本
    
  ```
  yum list docker-ce --showduplicates | sort -r
  ```
  
  ```
   * updates: mirrors.aliyun.com
  Loading mirror speeds from cached hostfile
  Loaded plugins: fastestmirror
   * extras: mirrors.aliyun.com
  docker-ce.x86_64    3:18.09.4-3.el7                            docker-ce-test   
  docker-ce.x86_64    3:18.09.4-3.el7                            docker-ce-stable 
  docker-ce.x86_64    3:18.09.4-2.1.rc1.el7                      docker-ce-test   
  docker-ce.x86_64    3:18.09.3-3.el7                            docker-ce-test   
  docker-ce.x86_64    3:18.09.3-3.el7                            docker-ce-stable 
  docker-ce.x86_64    3:18.09.3-2.1.rc1.el7                      docker-ce-test   
  docker-ce.x86_64    3:18.09.2-3.el7                            docker-ce-test   
  docker-ce.x86_64    3:18.09.2-3.el7                            docker-ce-stable 
  docker-ce.x86_64    3:18.09.1-3.el7                            docker-ce-test   
  docker-ce.x86_64    3:18.09.1-3.el7                            docker-ce-stable 
......
  ```
  返回的列表取决于启用了哪些存储库，并且制定了CentOS版本(例如el7表示centos7)。
  
  b.通过其完全限定的包名安装特定的版本，该包名是包名(docker-ce)加上版本字符串(第二列)，从第一个冒号(:)开始，直到第一个连字符(用连字符(-)分隔)。
  例如`docker-ce-18.09.4`
  
  `yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io`
  
  ```
  yum install docker-ce-18.09.4 docker-ce-cli-18.09.4 containerd.io
  ```
  
  3）启动docker
  
  ```
  service docker start
  ```
  
  4）运行hello-world镜像来验证docker-ce是否正确安装
  
  ```
  docker run hello-world
  ```
  
  ```
  Unable to find image 'hello-world:latest' locally
  latest: Pulling from library/hello-world
  1b930d010525: Pull complete 
  Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
  Status: Downloaded newer image for hello-world:latest
  
  Hello from Docker!
  This message shows that your installation appears to be working correctly.
  
  To generate this message, Docker took the following steps:
   1. The Docker client contacted the Docker daemon.
   2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
      (amd64)
   3. The Docker daemon created a new container from that image which runs the
      executable that produces the output you are currently reading.
   4. The Docker daemon streamed that output to the Docker client, which sent it
      to your terminal.
  
  To try something more ambitious, you can run an Ubuntu container with:
   $ docker run -it ubuntu bash
  
  Share images, automate workflows, and more with a free Docker ID:
   https://hub.docker.com/
  
  For more examples and ideas, visit:
   https://docs.docker.com/get-started/

  ```
  
  这条命令会下载一个测试映像并在容器中运行。当容器运行时，会打印一条消息并退出。打印以上信息表示docker-ce已经安装完毕。
  
### 2.6 卸载docker-ce

   1）卸载docker安装包：
   
   ```
   yum remove docker-ce
   ```

   2）主机上的映像、容器、卷或自定义配置文件并不会自动删除。
   删除所有图像、容器和卷:
   
   ```
   rm -rf /var/lib/docker
   ```