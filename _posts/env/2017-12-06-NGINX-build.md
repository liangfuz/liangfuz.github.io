---
layout: post
title: Centos yum 搭建Nginx
category: 环境搭建
tags: nginx
keywords: nginx
---

Nginx 是一个安装非常的简单、配置文件非常简洁（还能够支持perl语法）、Bug非常少的高性能HTTP和反向代理服务器
yum安装nginx也非常简单  
##第一步先获取安装包
`rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm`

##第二步安装并启动nignx
```
# yum install nginx
# service nginx start
Starting nginx:              [  OK  ]
```
##第三步进入浏览器，输入IP或者域名测试
如果看到以下输出则说明安装完成
```
Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.
For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.
Thank you for using nginx.
```
