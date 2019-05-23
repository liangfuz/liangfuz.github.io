---
layout: post
title: 5.docker Services服务编排（运用docker-compose）
category: docker
tags: docker
keywords: docker
---

## 1. 目标
   扩展docker应用并启用负载均衡

## 2. 什么是service
   service的的确确就是一个“生产容器”，一个只运行一个镜像的容器，例如使用哪个端口，一个容器拥有多少个运行节点等等。
   通过改变运行的容器实例数量可以达到扩展service的目的，以便使用更多的计算资源来为应用服务。
   
   在分布式应用程序中，应用程序的不同部分称为“service”。例如，如果我们一个视频共享站点，
   它可能包括一个数据存储的service、一个视频编解码的后台service、一个前端service，等等。
   
   要做到以上的功能很简单，我们只需要编写一个`docker-compose.yml`文件。
   
## 3.预备条件
   1.docker1.13或更高版本  
   2.Docker compose  
   3.掌握前面的章节内容  
   4.确保`docker run -p 4000:80 repo:tag`正常运行，如前面的`docker run -p 4000:80 friendlyhello`,打开`http://localhost:4000/`正常显示

### 4 编写`docker-compose.yml`文件
   ```
    version: "3"
    services:
      web:
        # replace username/repo:tag with your name and image details
        image: friendlyhello:latest
        deploy:
          replicas: 5
          resources:
            limits:
              cpus: "0.1"
              memory: 50M
          restart_policy:
            condition: on-failure
        ports:
          - "4000:80"
        networks:
          - webnet
    networks:
      webnet:
   ```
   
### 5 启动
   在执行`docker stack deploy`之前先执行`docker swarm init`命令  
   PS:这里我们涉及到了后面的命令，因为不执行这个的话会报错`this node is not a swarm manager.`
   启动命令：
   ```
   docker stack deploy -c docker-compose.yml getstartedlab
   ```
   这样我们就在一台主机上面运行了5个容器，加载的是同一个service镜像。
   执行`docker service ls`查看
   ```
   [root@localhost compose]# docker service ls
   ID                  NAME                MODE                REPLICAS            IMAGE                  PORTS
   zzdwlojws37n        getstartedlab_web   replicated          1/5                 friendlyhello:latest   *:4000->80/tcp
   ```
   如果上面使用的是`getstartedlab`，那么显示出来的就是`getstartedlab_web`
   
   也可以使用`docker stack services`加上应用名称查看
   ```
   [root@localhost compose]# docker stack services getstartedlab
   ID                  NAME                MODE                REPLICAS            IMAGE                  PORTS
   zzdwlojws37n        getstartedlab_web   replicated          5/5                 friendlyhello:latest   *:4000->80/tcp
   ```
   一个容器运行一个service称为一个task，每个task都有一个唯一的ID，我们在docker-compose.yml里面定义了几个拷贝就会有几个task。列出service对应的task：
   ```
   [root@localhost compose]# docker service ps getstartedlab_web
   ID                  NAME                      IMAGE                       NODE                    DESIRED STATE       CURRENT STATE                 ERROR                              PORTS
   s3mh4gjgikw9        getstartedlab_web.1       friendlyhello:latest        localhost.localdomain   Running             Running about a minute ago                                       
   lbb5keakzs7j        getstartedlab_web.2       friendlyhello:latest        localhost.localdomain   Running             Running about a minute ago                                       
   nl99uueulq2s        getstartedlab_web.3       friendlyhello:latest        localhost.localdomain   Running             Running about a minute ago                                       
   3uq7mtpzbny0        getstartedlab_web.4       friendlyhello:latest        localhost.localdomain   Running             Running about a minute ago                                       
   gtxx8ix6a3y7        getstartedlab_web.5       friendlyhello:latest        localhost.localdomain   Running             Running about a minute ago                                       
   ```
   列出所有容器：
   ```
   [root@localhost compose]# docker container ls -q
   9ae9a1b5396c
   2066432821f7
   75441d9e5ead
   4895dd5afae7
   0f390b88f394
   180cf775828c

   ```
   在浏览器多次访问`192.168.xxx.xxx:4000`能够看到不同的容器ID
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2019-05-05.png"/>
   
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2019-05-05-1.png"/>
   
### 5 应用扩展
   在`docker-compose.yml`文件里面，我们可以通过修改`replicas`的值来扩展应用。修改之后执行`docker stack deploy`命令：
   ```
   [root@localhost compose]# vim docker-compose.yml 
   [root@localhost compose]# docker stack deploy -c docker-compose.yml getstartedlab
   Updating service getstartedlab_web (id: zzdwlojws37nh23hgv75gp5kx)
   image friendlyhello:latest could not be accessed on a registry to record
   its digest. Each node will access friendlyhello:latest independently,
   possibly leading to different nodes running different

   [root@localhost compose]# docker stack services getstartedlab
   ID                  NAME                MODE                REPLICAS            IMAGE                  PORTS
   zzdwlojws37n        getstartedlab_web   replicated          6/6                 friendlyhello:latest   *:4000->80/tcp
   ```  
      
### 5 关闭应用
   ```
   [root@localhost compose]# docker stack rm getstartedlab
   Removing service getstartedlab_web
   Removing network getstartedlab_webnet
   
   [root@localhost compose]# docker swarm leave --force
   Node left the swarm.
   ``` 