---
layout: post
title: 服务器设置禁止root远程登录
category: 环境搭建
tags: root登陆
keywords: 禁止，root，登陆
---

有时候生产环境需要设置禁止远程root登录，我们需要设置禁止登录，用普通用户登录然后切换成root用户

### 先建一个普通用户
```
useradd test
```
给普通用户设置密码：
```
passwd test 123
```
删除用户：
```
userdel test
```
赋予root权限(普通用户不需要)
```
usermod -g root test
```

### 生产机器禁止ROOT远程SSH登录：
```
vim /etc/ssh/sshd_config
```
把
```
PermitRootLogin yes
```
改为
```
PermitRootLogin no
```
修改端口
找到 #Port 22 这一行，默认端口 22
```
#Port 22
```
我们可以把前面的#删除，然后把 22改为其它的端口
```
Port 100
```
重启sshd服务
```
#service sshd restart
```
远程管理用普通用户test登录，然后用 su root 切换到root用户拿到最高权限