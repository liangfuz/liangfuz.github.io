---
layout: post
title: centos6 yum搭建mysql
category: 环境搭建
tags: mysql
keywords: mysql
---

MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下产品。MySQL是最流行的关系型数据库管理系统之一

### 1.查看CentOS自带mysql是否已安装。
输入：
```
yum list installed | grep mysql
```
### 2.若有自带安装的mysql，如何卸载CentOS系统自带mysql数据库？
输入：
```
yum -y remove mysql-libs.x86_64 #若有多个依赖文件则依次卸载。
```
当结果显示为Complete！即卸载完毕。
### 3.查看yum库上的mysql版本信息(CentOS系统需要正常连接网络)。
输入：
```
yum -y list mysql*
```
### 4.使用yum安装mysql数据库。
输入：
```
yum -y install mysql-server mysql mysql-devel 
```
将mysql-server、mysql、mysql-devel都安装好，当结果显示为“Complete！”即安装完毕。
注：安装mysql只是安装了数据库，只有安装mysql-server才相当于安装了客户端。
### 5.查看刚安装mysql数据库版本信息。
输入：
```
rpm -qi mysql-server
```

启动/停止/重新启动/状态
```
service mysqld start
service mysqld stop
service mysqld restart
service mysqld status
pstree | grep mysqld //验证服务是否启动，比较少用；
```
### 6.Mysql字符集设置
输入查看字符集命令`SHOW VARIABLES LIKE 'character%';`
```
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | latin1                     |
| character_set_connection | latin1                     |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | latin1                     |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)
```
返回character_set_database和character_set_server的默认字符集是latin1。
修改/etc/my.cnf修改mysql字符集
```
vi /etc/my.cnf
```
my.cnf
```
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

```
在[mysqld]下面添加
```
default-character-set=utf8
character-set-server=utf8
```
如果没有[client]则添加
```
[client]
default-character-set=utf8
```
修改后的my.cnf
```
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
default-character-set=utf8
character-set-server=utf8

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
[client]
default-character-set=utf8
```
然后重启mysql
```
service mysqld restart
```
进入mysql查看字符集
```
mysql> SHOW VARIABLES LIKE 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

```
mysql字符集修改完成

### 7.用户名密码设置
初次安装mysql是root账户是没有密码的  
设置密码的方法如下
```
# mysql -uroot
mysql> set password for 'root'@'localhost' = password('mypasswd');
mysql> exit
```
### 8.允许mysql远程访问

mysql默认是不可远程访问的，可以使用以下三种方式修改:  
a、改表。
```
mysql -u root –p
mysql>use mysql;
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;
```
b、授权。

例如，root使用123456从任何主机连接到mysql服务器。
```
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
```
允许用户admin从ip为1.1.1.1的主机连接到mysql服务器，并使用123456作为密码
```
mysql>GRANT ALL PRIVILEGES ON *.* TO 'admin'@'1.1.1.1' IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql>FLUSH RIVILEGES
```
赋予任何主机访问数据的权限
```
mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION
mysql>FLUSH PRIVILEGES
```