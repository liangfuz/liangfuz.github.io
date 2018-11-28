---
layout: post
title: nginx 配置SSL https访问
category: 环境搭建
tags: ssl
keywords: ssl
---

最近想把自己做的网站在小程序里面做个入口，然后发现小程序的域名绑定需要https的，因为我的域名是在阿里云买的，所以决定去阿里云上面申请一个免费的SSL证书来使用

### 1.去阿里云上面申请证书。
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2018112801.png"/>

### 2.按照提示购买免费证书
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2018112802.png"/>

### 3.下载apache证书
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2018112803.png"/>

### 4.按照阿里云提示添加域名解析
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2018112805.png"/>

### 5.将下载好的crt和key文件上传到nginx目录下，我创建了一个文件夹cert，则上传到cert文件目录下
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2018112804.png"/>

### 6.修改nginx.conf配置

```
server {
       listen       443;
        server_name  www.xxx.com; #ssl绑定的域名
        ssl on;
         ssl_certificate cert/1558334_www.xxxx.com_public.crt; #申请的crt证书
         ssl_certificate_key cert/1558334_www.xxx.com.key; #申请的crt证书密码
         ssl_session_timeout 5m;
         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
         ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
        ssl_prefer_server_ciphers on;
                location / {
                        root /.......
                }

       
   }
```

### 7.测试nginx配置无误然后重新加载nginx
`nginx -t`

`service nginx reload`

```
[root@iZbp1h9mcsf8qws27z5d6tZ nginx]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@iZbp1h9mcsf8qws27z5d6tZ nginx]# service nginx reload
Reloading nginx:                                           [  OK  ]
[root@iZbp1h9mcsf8qws27z5d6tZ nginx]# 

```

### 8.测试ssl是否已经生效

浏览器访问自己的域名，https开头能正常访问说明配置成功！

<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/2018112806.png"/>
