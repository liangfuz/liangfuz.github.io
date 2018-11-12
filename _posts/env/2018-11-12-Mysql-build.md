---
layout: post
title: centos7 yum搭建mysql
category: 环境搭建
tags: mysql
keywords: mysql
---

CentOS7默认数据库是mariadb, 但是 好多用的都是mysql ，但是CentOS7的yum源中默认是没有mysql的。

上一篇安装的是centos6的但是我想在centos7中安装mysql，yum安装是最简单的，决定用yum。

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
### 3.下载mysql的repo源 这个安装的mysql5.7.20
输入：
```
wget http://repo.mysql.com/mysql57-community-release-el7-8.noarch.rpm 
rpm -ivh mysql57-community-release-el7-8.noarch.rpm 
```
### 4.使用yum安装mysql数据库。
输入：
```
yum -y install mysql-server 
```
将mysql-server、mysql、mysql-devel都安装好，当结果显示为“Complete！”即安装完毕。
注：安装mysql只是安装了数据库，只有安装mysql-server才相当于安装了客户端。
### 5.查看刚安装mysql数据库版本信息。
输入：
```
rpm -qi mysql-server
```
之后的操作和上一篇一样

PS:第一次可能遇到登录密码随机的问题
生成随机密码

`grep "password" /var/log/mysqld.log`
```
2018-11-12T07:57:43.270119Z 1 [Note] A temporary password is generated for root@localhost: -;utlZ:BW1p(
2018-11-12T07:58:04.085456Z 2 [Note] Access denied for user 'root'@'localhost' (using password: NO)
2018-11-12T07:58:09.484134Z 3 [Note] Access denied for user 'root'@'localhost' (using password: YES)
2018-11-12T07:58:15.117255Z 4 [Note] Access denied for user 'root'@'localhost' (using password: NO)
2018-11-12T07:58:55.375310Z 5 [Note] Access denied for user 'root'@'localhost' (using password: NO)
```
其中`-;utlZ:BW1p(`是密码，登录之后重置密码

`alter user 'root'@'localhost' identified by 'Root@2018';`

密码有规则，可能是大小写特殊字符什么的