---
layout: post
title: 6.docker Swarms集群搭建
category: docker
tags: docker
keywords: docker
---

## 1. 目标
   在前面的章节我们已经掌握了如何打包运行，编排、扩展服务。
   本节内容将介绍如何将我们的应用部署到集群。使用docker的方式部署集群将多应用运行在多机器，多容器上面，这种方式我们叫做swarm集群。

## 2.预备条件
   1.docker1.13或更高版本  
   2.Docker compose  
   3.掌握前面的章节内容  
   4.确保`docker run -p 4000:80 repo:tag`正常运行，如前面的`docker run -p 4000:80 friendlyhello`,打开`http://localhost:4000/`正常显示
   5.上一章节的`docker-compose.yml`拷贝
   6.准备两台linux虚拟机（如果使用VmWare的话可以将虚拟机下面的磁盘（vmdk文件）复制，这样就能得到两台一样的环境的VM linux主机，
   省去重新安装docker等一系列环境的麻烦）
   
## 3. 什么是Swarm集群
   `swarm`是一群运行着docker的机器加入在一起形成的集群。当集群搭建好之后，我们还是像之前一样执行docker的命令，只不过这些命令都被`swarm manager`执行在集群上。
   这些机器可以是物理机也可以是虚拟机，我们称之为节点`nodes`。  
   `Swarm managers`可以使用多种策略来运行容器，比如`emptiest node`，这个指令会将容器运行在利用率最低的机器上。或者`global`，这个确保每台机器只
   运行一个特定的容器的实例。做到上面说的我们只需要在`docker-compose.yml`文件里面设置好`swarm manager`的策略就可以了。  
   在swarm集群中只有一台机器作为`swarm manager`，swarm manager可以执行命令，也可以授权其他机器加入swarm成为`workers`。workers只能
   作为容量扩展，不能和其他的机器交流。
   
### 4 搭建Swarm集群
   一个swarm集群是由多个物理或者虚拟的节点组成。最简单的是执行`docker swarm init`使得当前机器开启swarm模式，同时当前机器成为swarm manager
   节点，然后在其他的节点执行`docker swarm join`使其他的节点加入swarm成为workers。下面主要描述的是如何在两台虚拟linux系统上面搭建swarm集群。
 
### 5 创建cluster
#### 5.1 启动swarm manager节点
   按照2.6的操作我们现在拥有两台虚拟机，使用第一台作为manager，执行`docker swarm init --advertise-addr <ip>`命令：
   ```
   [root@localhost ~]# docker swarm init --advertise-addr 192.168.10.152
   Swarm initialized: current node (t8jekqeafyth52r593rqw25gv) is now a manager.
   
   To add a worker to this swarm, run the following command:
   
       docker swarm join --token SWMTKN-1-4o5akm4y891j6qlop54l5lx827sfc0nf721mg37ggrx93gqn7m-3meu1adz1yyiderp5xzj78z8q 192.168.10.152:2377
   
   To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
   ```
   ps:为了方便查看，我们把第一台设备的hostname修改为master，第二台修改为worker1，然后重新启动
   ```
   [root@localhost ~]# vim /etc/hostname 
   [root@localhost ~]# reboot
   ```
#### 5.2 加入worker节点
   按照上面说的，在另外一个节点上面执行`docker swarm join-token <token> <ip>:2377`命令：
   ```
   [root@localhost ~]# docker swarm join --token SWMTKN-1-4o5akm4y891j6qlop54l5lx827sfc0nf721mg37ggrx93gqn7m-3meu1adz1yyiderp5xzj78z8q 192.168.10.152:2377
   This node joined a swarm as a worker.
   ```
   如果报错的话就有可能是开启了防火墙
   ```
   [root@localhost ~]# docker swarm join --token SWMTKN-1-4o5akm4y891j6qlop54l5lx827sfc0nf721mg37ggrx93gqn7m-3meu1adz1yyiderp5xzj78z8q 192.168.10.152:2377
   Error response from daemon: rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: Error while dialing dial tcp 192.168.10.152:2377: connect: no route to host"
   ```
   关闭防火墙
   ```
   [root@localhost ~]# firewall-cmd --state
   running
   [root@localhost ~]# systemctl stop firewalld.service
   [root@localhost ~]# firewall-cmd --state
   not running
   ```
#### 5.3 查看节点
   在第一台机上面执行`docker node ls`命令查看节点
   ```
   [root@master compose]# docker node ls
   ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
   spcwv3gpf3h2tnf0a63pasm2b *   master              Ready               Active              Leader              18.09.4
   drzccu7hwgb3a7toqc23f9l4c     worker1             Ready               Active                                  18.09.4
   ```
#### 5.4 退出swarm
   如果想要退出swarm执行`docker swarm leave`命令
   
### 6 在swarm集群上面部署app
#### 6.1 设置环境变量  
   在第一个节点上面设置环境变量：
   ```
   export DOCKER_TLS_VERIFY=1
   export DOCKER_HOST=tcp://192.168.10.152:2377
   #export DOCKER_CERT_PATH="/Users/sam/.docker/machine/machines/myvm1"
   export DOCKER_MACHINE_NAME=localhost.localdomain #hostname主机名
   ```
   立即生效
   ```
   [root@localhost ~]# vim /etc/hosts
   [root@localhost ~]# source /etc/profile
   ```
#### 6.2 在swarm manager上面部署app
   至此，我们拥有了swarm manager主机，我们可以使用`docker stack deploy -c docker-compose.yml getstartedlab`命令来
   部署之前的friendlyhello app，而后使用`docker stack ps getstartedlab`查看就可以发现原本单机运行6个实例的被分别运行在
   swarm集群的master和worker1上面
   
   ```
   [root@master compose]# docker stack deploy -c docker-compose.yml getstartedlab
   Creating network getstartedlab_webnet
   Creating service getstartedlab_web
   [root@master compose]# docker stack ps getstartedlab
   ID                  NAME                  IMAGE                  NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
   m02zc2b1dmw7        getstartedlab_web.1   friendlyhello:latest   worker1             Running             Running 10 seconds ago                       
   85jqdzzfi32v        getstartedlab_web.2   friendlyhello:latest   master              Running             Running 12 seconds ago                       
   ybjz6iz4nnu1        getstartedlab_web.3   friendlyhello:latest   worker1             Running             Running 10 seconds ago                       
   z6xxcuacf5ya        getstartedlab_web.4   friendlyhello:latest   master              Running             Running 14 seconds ago                       
   asuejwsdjy8t        getstartedlab_web.5   friendlyhello:latest   worker1             Running             Running 11 seconds ago                       
   rs6ccjya8b63        getstartedlab_web.6   friendlyhello:latest   master              Running             Running 13 seconds ago
   ```
#### 6.3 浏览器分别访问`192.168.10.152:4000`和`192.168.10.153:4000`
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190516.png"/>
   
#### 6.4 docker swarm 工作路由
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190516-1.png"/>
   
#### 6.5 关闭
   `docker stack rm getstartedlab`
   ```
   [root@master build]# docker stack rm getstartedlab
   Removing service getstartedlab_web
   Removing network getstartedlab_webnet
   [root@master build]# docker stack ps getstartedlab
   ID                  NAME                          IMAGE                       NODE                DESIRED STATE       CURRENT STATE                     ERROR               PORTS
   qck6u4fv9idw        nn1y48taebpqf1j7ar6zy23qx.2   root/friendlyhello:latest   worker1             Remove              Shutdown less than a second ago                       
   mnyazxnob78r        nn1y48taebpqf1j7ar6zy23qx.3   root/friendlyhello:latest   worker1             Remove              Preparing 3 seconds ago                               
   yttg4fmm0ww6        nn1y48taebpqf1j7ar6zy23qx.4   root/friendlyhello:latest   worker1             Remove              Shutdown less than a second ago                       
   8ac9uzkg9yfl        nn1y48taebpqf1j7ar6zy23qx.5   root/friendlyhello:latest   worker1             Remove              Shutdown less than a second ago
   ```