---
layout: post
title: Mongodb启动child process failed, exited with error number 1
category: 环境搭建
tags: Mongodb
keywords: Mongodb error
---

## linux重启之后mongodb启动报错
    ```
    about to fork child process, waiting until server is ready for connections.
    forked process: 15246
    ERROR: child process failed, exited with error number 1
    To see additional information in this output, start without the "--fork" option.

    ```
   这个原因是由于MongoDB的pid文件被linux重启清理掉了，需要修改/etc/mongo.conf文件下pid文件目录
   `pidFilePath: /xxxxx/mongodb/mongod.pid`修改到一个固定的位置，例如`/mongodb/mongod.pid`
   下面是具体的步骤：
   1.创建空的MongoDB pid文件
   ```
   cd /
   mkdir mongodb
   cd mongodb
   vi mongod.pid #保存空文件
   ```
   2.修改MongoDB配置文件
   ```
   [root@localhost ~]# whereis mongod
   mongod: /usr/bin/mongod /etc/mongod.conf /etc/mongod.conf2 /usr/share/man/man1/mongod.1.gz
   vi /etc/mongod.conf #修改pidFilePath为上面保存的文件 pidFilePath=/mongodb/mongod.pid
   ```
   3.重新启动MongoDB
   ```
   [root@localhost mongodb]# vi /etc/mongod.conf
   [root@localhost mongodb]# /usr/bin/mongod -f /etc/mongod.conf
   about to fork child process, waiting until server is ready for connections.
   forked process: 15479
   child process started successfully, parent exiting
   ```