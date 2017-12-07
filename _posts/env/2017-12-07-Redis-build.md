---
layout: post
title: Linux下搭建redis
category: 环境搭建
tags: redis
keywords: redis,centos,linux
---

Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。从2010年3月15日起，Redis的开发工作由VMware主持。
redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、 list(链表)、set(集合)、zset(sorted set –有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原 子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的 把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

### 一、下载安装：
##### 1）下载文件到 {path} 目录下
```
cd {path}
wget http://download.redis.io/releases/redis-2.8.13.tar.gz
```
##### 2）解压文件
```
tar zxvf redis-2.8.13.tar.gz
```
##### 3）切换目录到 redis-2.8.13 目录下
```
cd {path}/redis-2.8.13
```
##### 4）执行make命令，最后几行的输出结果
```
make
Hint: To run ‘make test’ is a good idea
make[1]: Leaving directory {path}/redis-2.8.13/src
```
##### 5）按照提示执行：
```
make test
```
##### 6）提示：（没有提示的话直接跳到第8步make install完成安装）
```
You need tcl 8.5 or newer in order to run the Redis test
make: *** [test] Error 1
```
##### 7）可以使用命令安装
```
yum install tcl 
```
##### 8）之后再执行
```
make install
```
提示成功，redis已经安装完毕
##### 9）启动Redis服务。
```
 redis-server   redis.conf
```
##### 10）然后用客户端测试一下是否启动成功。
```redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```
### 二、客户端操作
下面的操作对配置的修改都是重启就无效的，要修改配置并永久生效需要修改redis.conf文件，后面讲述  
#### 1）连接客户端（没有修改配置文件的话默认端口6379）
```
redis-cli
```
#### 2）查看当前的密码：
```
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> config get requirepass
1) "requirepass"
2) (nil)
```
显示密码是空的，
然后设置密码：
```
redis 127.0.0.1:6379> config set requirepass test123
OK
```
再次查询密码：
```
redis 127.0.0.1:6379> config get requirepass
(error) ERR operation not permitted
```
此时报错了！
现在只需要密码认证就可以了。
```
redis 127.0.0.1:6379> auth test123
OK
```
再次查询密码：
```
redis 127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "test123"
```
密码已经得到修改。
当到了可以重启redis的时候 由于配置参数已经修改 所以密码会自动生效。
要是配置参数没添加密码 那么redis重启密码将相当于没有设置。
 
#### 3）如何登录有密码的redis？
a.在登录的时候 密码就输入
```
redis-cli -p 6379 -a test123
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "test123"
```
b.先登录再验证：
```
redis-cli -p 6379
redis 127.0.0.1:6379>
redis 127.0.0.1:6379> auth test123
OK
redis 127.0.0.1:6379> config get requirepass
1) "requirepass"
2) "test123"
redis 127.0.0.1:6379>
```
### 三、配置文件修改
redis的配置在redis目录下的redis.conf（<span style="color:red">记得修改配置的时候前面的空格要去掉，不然会有问题</span>）
#### 1）修改密码
打开redis.conf找到
```
vi redis.conf
```

```
#requirepass foobared
```
改成
```
requirepass {password}
```
#### 2）设置以守护进程运行
如果不设置的话启动redis服务之后ctrl+c会关闭redis
```
#daemonize no
```
改成
```
daemonize yes
```
修改端口
```
port 6379
```
#### 3）redis主从设置
在同一台服务器上坐测试需要修改port
1.复制一份conf文件
```
cp redis.conf redis-slave.conf
```
2.修改端口
```
port 6379
```
改成
```
port 63791
```
3.修改绑定master的IP，同台服务器可设置为127.0.0.1
```
#bind 127.0.0.1
```
改成
```
bind 127.0.0.1
```
4.设置slave配置
```
# slaveof <masterip> <masterport> 

```
改成
```
slaveof 127.0.0.1 6379

```
5.当master有密码的时候 配置slave 的时候 相应的密码参数也得相应的配置好。不然slave 是无法进行正常复制的。
相应的参数是：
```
#masterauth
```
改成
```
masterauth  mstpassword
```

配置了密码，启动的时候需要带上conf文件  redis-server redis.conf
修改了端口，启动命令修改 redis-cli -p 8888
配置修改完需要重启才能生效
7.启动slave redis
```
# redis-server redis-slave.conf 
```
8.查看redis服务进程
```
# ps ax|grep redis
18605 ?        Ssl    0:07 redis-server *:6379    
18784 ?        Ssl    0:00 redis-server 127.0.0.1:63791 
18789 pts/0    S+     0:00 grep redis
```
9.打开redis-cli测试主从
```
[root@iZ28nujjnwqZ redis-2.8.13]# redis-cli
127.0.0.1:6379> set testslave mast-slave
OK
127.0.0.1:6379> get testslave
"mast-slave"
127.0.0.1:6379> 
[root@iZ28nujjnwqZ redis-2.8.13]# redis-cli -p 63791
127.0.0.1:63791> get testslave
"mast-slave"
```
slave设置为只读的话不能设值
```
127.0.0.1:63791> set testslave sss
(error) READONLY You can't write against a read only slave.
```
