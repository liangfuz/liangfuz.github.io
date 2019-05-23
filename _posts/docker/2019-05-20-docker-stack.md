---
layout: post
title: 7.docker Stack使用
category: docker
tags: docker
keywords: docker
---


## 1.预备条件
   1.docker1.13或更高版本  
   2.Docker compose  
   3.掌握前面的章节内容  
   4.确保`docker run -p 4000:80 repo:tag`正常运行，如前面的`docker run -p 4000:80 friendlyhello`,打开`http://localhost:4000/`正常显示
   5.上一章节的`docker-compose.yml`拷贝
   6.像上一章节一样拥有两台虚拟主机
   
## 2. 什么是Stack
   `stack`是docker分布式应用程序层次结构的顶部，由一组相互关联的,可共享依赖关系，并且可以一起编排和扩展的`service`构成。  
   在之前我们已经使用过`docker stack deploy`来部署我们的`service`，但那是一个单独的服务运行在单台主机上面，而一般在生产环境上面
   我们都是使多个服务彼此关联，并在多台机器上运行。
   
## 3. 部署stack服务
### 3.1 编写`docker-compose.yml`文件

   ```
   version: "3"
   services:
     web:
       # replace username/repo:tag with your name and image details
       image: username/repo:tag
       deploy:
         replicas: 5
         restart_policy:
           condition: on-failure
         resources:
           limits:
             cpus: "0.1"
             memory: 50M
       ports:
         - "80:80"
       networks:
         - webnet
     visualizer:
       image: dockersamples/visualizer:stable
       ports:
         - "8080:8080"
       volumes:
         - "/var/run/docker.sock:/var/run/docker.sock"
       deploy:
         placement:
           constraints: [node.role == manager]
       networks:
         - webnet
   networks:
     webnet:
   ```
   和之前的相比，这里新增了一个视图`visualizer`web服务，和一个可以让`visualizer`访问主机的docker socket的`volumes`
   ,另外还有一个`placement`确保这个只运行在swarm manager而不是worker上面。  
   后面有空会介绍一下`volumes`和`placement constraints`
   
### 3.2 部署
   执行`docker stack deploy -c docker-compose.yml getstartedlab`命令
   ```
   [root@master stack]# docker stack deploy -c docker-compose.yml getstartedlab
   Creating network getstartedlab_webnet
   Creating service getstartedlab_web
   Creating service getstartedlab_visualizer
   ```
   查看运行情况`docker stack ps getstartedlab`
   ```
   [root@master stack]# docker stack ps getstartedlab
   ID                  NAME                             IMAGE                             NODE                DESIRED STATE       CURRENT STATE                 ERROR                              PORTS
   ww0ayglg8e6x        getstartedlab_visualizer.1       dockersamples/visualizer:stable   master              Running             Preparing 12 seconds ago                                         
   npghje1cj7jw         \_ getstartedlab_visualizer.1   dockersamples/visualizer:stable   master              Shutdown            Rejected 12 seconds ago       "No such image: dockersamples/…"   
   cun6e8dttiw3         \_ getstartedlab_visualizer.1   dockersamples/visualizer:stable   master              Shutdown            Rejected 28 seconds ago       "No such image: dockersamples/…"   
   0d5jykci2ld1         \_ getstartedlab_visualizer.1   dockersamples/visualizer:stable   master              Shutdown            Rejected 44 seconds ago       "No such image: dockersamples/…"   
   k9ojoynicezp         \_ getstartedlab_visualizer.1   dockersamples/visualizer:stable   master              Shutdown            Rejected about a minute ago   "No such image: dockersamples/…"   
   vf78h21rp0ae        getstartedlab_web.1              friendlyhello:latest              master              Running             Running 2 minutes ago                                            
   tcq124baj9ha        getstartedlab_web.2              friendlyhello:latest              worker1             Running             Running 2 minutes ago                                            
   7nt6yn2s7m6r        getstartedlab_web.3              friendlyhello:latest              worker1             Running             Running 2 minutes ago                                            
   oqx766e5v3gn        getstartedlab_web.4              friendlyhello:latest              master              Running             Running 2 minutes ago                                            
   oz5qlib0fyz7        getstartedlab_web.5              friendlyhello:latest              worker1             Running             Running 2 minutes ago                                            
   ```
   `"No such image: dockersamples/…"`Error解决办法：  
   执行`docker pull dockersamples/visualizer:stable`获取image
   ```
   [root@master stack]# docker pull dockersamples/visualizer:stable
   Error response from daemon: Get https://registry-1.docker.io/v2/: x509: certificate has expired or is not yet valid
   ```
   报错`certificate has expired or is not yet valid`的话有可能是系统时间不对，需要同步时间
   ```
   [root@master stack]# date
   Tue May 23 06:24:31 EDT 2017
   [root@master stack]# ntpdate cn.pool.ntp.org
   -bash: ntpdate: command not found
   [root@master stack]# yum install -y ntpdate
   [root@master stack]# ntpdate cn.pool.ntp.org
   ```
   之后再次拉取镜像
   ```
   [root@master stack]# docker pull dockersamples/visualizer:stable
   stable: Pulling from dockersamples/visualizer
   Digest: sha256:bc680132f772cb44062795c514570db2f0b6f91063bc3afa2386edaaa0ef0b20
   Status: Image is up to date for dockersamples/visualizer:stable
   ```
   重新部署应用`docker stack deploy -c docker-compose.yml getstartedlab`
   ```
   [root@master stack]# docker stack deploy -c docker-compose.yml getstartedlab
   Creating network getstartedlab_webnet
   Creating service getstartedlab_web
   Creating service getstartedlab_visualizer
   [root@master stack]# docker stack ps getstartedlab
   ID                  NAME                         IMAGE                             NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
   ibr3w508l2up        getstartedlab_visualizer.1   dockersamples/visualizer:stable   master              Running             Running 4 seconds ago                         
   r31zyajuiaqc        getstartedlab_web.1          friendlyhello:latest              worker1             Running             Preparing 6 seconds ago                       
   lg1071xid4nw        getstartedlab_web.2          friendlyhello:latest              worker1             Running             Preparing 6 seconds ago                       
   g5tos43996la        getstartedlab_web.3          friendlyhello:latest              master              Running             Preparing 8 seconds ago                       
   owc8ef8c27fv        getstartedlab_web.4          friendlyhello:latest              worker1             Running             Preparing 6 seconds ago                       
   zquir8ai8fx7        getstartedlab_web.5          friendlyhello:latest              master              Running             Preparing 8 seconds ago
   ```
### 3.3 浏览器查看可视化界面
   在浏览器地址栏输入`192.168.10.152:8080`查看可视化视图  
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190523.png"/>
   
### 3.4 持久化数据
   继续上面的步骤，我们来尝试添加一个redis缓存数据库来保存应用数据。  
   在刚刚的`docker-compose.yml`文件里面增加redis的配置：  
   ```
   version: "3"
   services:
     web:
       # replace username/repo:tag with your name and image details
       image: username/repo:tag
       deploy:
         replicas: 5
         restart_policy:
           condition: on-failure
         resources:
           limits:
             cpus: "0.1"
             memory: 50M
       ports:
         - "80:80"
       networks:
         - webnet
     visualizer:
       image: dockersamples/visualizer:stable
       ports:
         - "8080:8080"
       volumes:
         - "/var/run/docker.sock:/var/run/docker.sock"
       deploy:
         placement:
           constraints: [node.role == manager]
       networks:
         - webnet
     redis:
       image: redis
       ports:
         - "6379:6379"
       volumes:
         - "/home/docker/data:/data"
       deploy:
         placement:
           constraints: [node.role == manager]
       command: redis-server --appendonly yes
       networks:
         - webnet
   networks:
     webnet:
   ```
   docker库里面有redis的官方镜像，所以我们只需要添加镜像`redis`就可以了，不需要附加`username/repo`。  
   容器里面的数据`/data`随容器的销毁会一并销毁，所以我们需要将数据卷挂载出来，例如上面，我们将容器里面的`/data`挂载到宿主机的
   `/home/docker/data`里面，所以我们需要新建目录`mkdir /home/docker/data -p`
   ```
   [root@master stack]# mkdir /home/docker/data -p
   ```
   而后重新部署、查看应用：
   ```
   [root@master stack]# docker stack deploy -c docker-compose.yml getstartedlab
   Creating network getstartedlab_webnet
   Creating service getstartedlab_redis
   Creating service getstartedlab_web
   Creating service getstartedlab_visualizer
   [root@master stack]# docker stack ps getstartedlab
   ID                  NAME                          IMAGE                             NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
   nt8oz3rkz2h2        getstartedlab_visualizer.1    dockersamples/visualizer:stable   master              Running             Running 1 second ago                          
   s2g2hncgbmpw        getstartedlab_web.1           friendlyhello:latest              master              Running             Preparing 4 seconds ago                       
   fpeggtrky71k        getstartedlab_redis.1         redis:latest                      master              Running             Running 5 seconds ago                         
   nc39u6a19bpr        pgw4idn3317bqu0s4a5m79b0o.1   friendlyhello:latest              worker1             Remove              Starting 6 minutes ago                        
   dt84obucy2vk        getstartedlab_web.2           friendlyhello:latest              worker1             Running             Preparing 2 seconds ago                       
   pgitof153g3p        zeimsj5fqitvqs0m3xjhetklq.2   friendlyhello:latest              worker1             Remove              Starting 3 minutes ago                        
   5sa4qj0glzrd        getstartedlab_web.3           friendlyhello:latest              master              Running             Preparing 4 seconds ago                       
   yirzi7n40xsz        zeimsj5fqitvqs0m3xjhetklq.3   friendlyhello:latest              worker1             Remove              Starting 3 minutes ago                        
   jkv0kn6fjbkz        pgw4idn3317bqu0s4a5m79b0o.3   friendlyhello:latest              worker1             Remove              Starting 6 minutes ago                        
   m3cupgojoy1f        getstartedlab_web.4           friendlyhello:latest              worker1             Running             Preparing 2 seconds ago                       
   k2dags8lokpo        getstartedlab_web.5           friendlyhello:latest              worker1             Running             Preparing 2 seconds ago                       
   cdjtqal0hlvu        zeimsj5fqitvqs0m3xjhetklq.5   friendlyhello:latest              worker1             Remove              Starting 3 minutes ago                        
   eezbpc1xqgtt        pgw4idn3317bqu0s4a5m79b0o.5   friendlyhello:latest              worker1             Remove              Starting 6 minutes ago                        
   [root@master stack]# docker service ls
   ID                  NAME                       MODE                REPLICAS            IMAGE                             PORTS
   k3x9sjywiwmp        getstartedlab_redis        replicated          1/1                 redis:latest                      *:6379->6379/tcp
   yi027pcc6o07        getstartedlab_visualizer   replicated          1/1                 dockersamples/visualizer:stable   *:8080->8080/tcp
   dz6y6l63hc0o        getstartedlab_web          replicated          2/5                 friendlyhello:latest              *:80->80/tcp
   ```
   而后在浏览器里面输入`http://192.168.10.152/`、`http://192.168.10.152:8080`分别查看`hello`和`visualizer`服务
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190523-2.png"/>  
   
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190523-1.png"/>
     
   可以看到`hello`服务的redis已经正常运作，且`visualizer`服务也多了一个redis服务，`/home/docker/data`下面也多了redis的持久化数据
   ```
   [root@master stack]# cd /home/docker/data/
   [root@master data]# ll
   total 4
   -rw-r--r--. 1 polkitd input 167 May 23 07:39 appendonly.aof
   ```
   至此，docker的镜像，容器，打包运行，服务编排，负载集群等基本操作都过了一遍，
   下一篇将运用docker对SpringCloud微服务部署做一个实战训练，争取对以上的内容做到充分熟练的运用。