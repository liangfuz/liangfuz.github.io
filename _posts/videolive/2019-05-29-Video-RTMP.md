---
layout: post
title: Nginx+RTMP搭建流媒体直播服务器
category: 视频直播
tags: videolive
keywords: rtmp,nginx
---

RTMP是Real Time Messaging Protocol（实时消息传输协议）的首字母缩写。该协议基于TCP，是一个协议族，包括RTMP基本协议及RTMPT/RTMPS/RTMPE等多种变种。RTMP
是一种设计用来进行实时数据通信的网络协议，主要用来在Flash/AIR平台和支持RTMP协议的流媒体/交互服务器之间进行音视频和数据通信。支持该协议的软件包括Adobe Media Server/Ultrant Media 
Server/red5等。
#### 阿里的直播流程图    
<img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190529.png"/>  
这里只做一个简单的直播搭建的详细步骤记录，不涉及CDN以及低延迟等技术的实施（<a href="https://help.aliyun.com/product/29949
.html">阿里云</a>已经有成熟的方案，只不过价格不菲，有钱的土豪可以直接购买，然后进行开发对接）  
我们这里使用OBS软件模拟推流，VLC播放模拟拉流，相当于用obs直播上传PC显示器内容到Nginx，然后用VLC播放


### 1. nginx rtmp模块集成
#### 1.1 下载<a href="http://nginx.org/en/download.html">nginx安装包</a>
   下载nginx安装包并解压
   ```
   wget http://nginx.org/download/nginx-1.10.3.tar.gz
   tar zxvf nginx-1.10.3.tar.gz
   ```
#### 1.2 下载nginx-rtmp-module
   下载<a href="http://arut.github.io/nginx-rtmp-module/">rtmp module</a>并上传到linux服务器
   ```
   tar zxvf arut-nginx-rtmp-module-v1.2.1-0-g791b613.tar.gz
   mv arut-nginx-rtmp-module-791b613/ nginx-rtmp-module
   ```
#### 1.3 编译nginx
   在nginx目录下面执行make命令，添加rtmp模块
   ```
   [root@master nginx-1.10.3]# ./configure --add-module=/opt/nginx-rtmp-module
   ...
   checking for OS
    + Linux 3.10.0-957.el7.x86_64 x86_64
   checking for C compiler ... not found
   
   ./configure: error: C compiler cc is not found
   ```
   提示没有安装gcc, 执行以下命令安装
   ```
   [root@master nginx-1.10.3]# yum install gcc gcc-c++
   ```
   重新编译之后报错提示增加`--with-http_ssl_module --without-http_rewrite_module`参数
   ```
   ./configure: error: the HTTP rewrite module requires the PCRE library.
   You can either disable the module by using --without-http_rewrite_module
   option, or install the PCRE library into the system, or build the PCRE library
   statically from the source with nginx by using --with-pcre=<path> option.
   ```
   重新编译
   ```
   [root@master nginx-1.10.3]# ./configure --add-module=/opt/nginx-rtmp-module  --with-http_ssl_module --without-http_rewrite_module
   ...
   ./configure: error: SSL modules require the OpenSSL library.
   You can either do not enable the modules, or install the OpenSSL library
   into the system, or build the OpenSSL library statically from the source
   with nginx by using --with-openssl=<path> option.
   ```
   提示没有安装ssl，安装ssl
   ```
   [root@master nginx-1.10.3]# yum -y install openssl openssl-devel
   ```
   再次编译
   ```
   [root@master nginx-1.10.3]# ./configure --add-module=/opt/nginx-rtmp-module  --with-http_ssl_module --without-http_rewrite_module
   ...
   Configuration summary
     + PCRE library is not used
     + using system OpenSSL library
     + md5: using OpenSSL library
     + sha1: using OpenSSL library
     + using system zlib library
   
     nginx path prefix: "/usr/local/nginx"
     nginx binary file: "/usr/local/nginx/sbin/nginx"
     nginx modules path: "/usr/local/nginx/modules"
     nginx configuration prefix: "/usr/local/nginx/conf"
     nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
     nginx pid file: "/usr/local/nginx/logs/nginx.pid"
     nginx error log file: "/usr/local/nginx/logs/error.log"
     nginx http access log file: "/usr/local/nginx/logs/access.log"
     nginx http client request body temporary files: "client_body_temp"
     nginx http proxy temporary files: "proxy_temp"
     nginx http fastcgi temporary files: "fastcgi_temp"
     nginx http uwsgi temporary files: "uwsgi_temp"
     nginx http scgi temporary files: "scgi_temp"

   ```
   而后执行make以及install安装nginx
   ```
   [root@master nginx-1.10.3]# make && make install
   make -f objs/Makefile
   make[1]: Entering directory `/opt/nginx-1.10.3'
   ...
   test -d '/usr/local/nginx/logs' \
   	|| mkdir -p '/usr/local/nginx/logs'
   make[1]: Leaving directory `/opt/nginx-1.10.3'
   ```
   安装完毕之后找到nginx安装目录启动
   ```
   [root@master sbin]# cd /usr/local/nginx/sbin/
   [root@master sbin]# ./nginx 
   ```
   浏览器访问提示成功启动
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190529-1.png"/>  
   
   直播Nginx服务搭建完毕。
   
### 2. nginx rtmp配置
   编辑`nginx.conf`添加rtmp配置
   ```
   rtmp {
   server {
   listen 1935;
   chunk_size 4000;
   application mylive {
   live on;
   record all;
   record_path /home/live_record;
   record_max_size 200M;
   hls on;
   hls_path /home/hls;
   hls_fragment 1s;
   hls_playlist_length 5;
   allow play all;
   }
   application live{
   live on;
   }
   }
   }

   ```
   ```
   [root@master sbin]# cd /usr/local/nginx/conf/
   [root@master conf]# vim nginx.conf
   ```
   添加后
   ```
   #user  nobody;
   worker_processes  1;
   
   #error_log  logs/error.log;
   #error_log  logs/error.log  notice;
   #error_log  logs/error.log  info;
   
   #pid        logs/nginx.pid;
   
   
   events {
       worker_connections  1024;
   }
   
   rtmp {
   server {
   listen 1935;
   chunk_size 4000;
   application mylive {
   live on;
   record all;
   record_path /home/live_record;
   record_max_size 200M;
   hls on;
   hls_path /home/hls;
   hls_fragment 1s;
   hls_playlist_length 5;
   allow play all;
   }
   application live{
   live on;
   }
   }
   }

   
   http {
       include       mime.types;
       default_type  application/octet-stream;
   
       #log_format  main  '$remote_addr - $remote_user [
   ```
   重新加载配置`/usr/local/nginx/sbin/nginx -s reload`
   
### 3. OBS推流，VLC拉流
   下载<a href="http://www.obsapp.com/obsdownload/">`OBS软件`</a>和`VLC播放软件`按下图配置好  
   添加来源为显示器获取
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190529-2.png"/>  
   广播设定FMS URL 为`rtmp://xxxx:1935/live/`
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190529-3.png"/>  
   点击开始串流开启直播推流  
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190529-4.png"/>  
   启动VLC，媒体->打开网络串流，输入拉流地址`rtmp://xxxx:1935/live`
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190529-5.png"/>
   
   效果图（左边OBS模拟的是上传，右边VLC模拟的是播放客户端，因为我是在同一台电脑上演示的，所以效果糅在一起了）  
   
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/rtmp3.gif"/>
   
   至此，一个简易的直播服务就搭建完毕了。
   
### 4. 多个房间
   开启多个房间或者说多个频道只需要在OBS上面修改`播放路径/串码流`,然后在VLC的播放路径作相应的修改就可以了
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190605.png"/>
   
   <img src="http://github-blog.oss-cn-shenzhen.aliyuncs.com/20190605-1.png"/>
   
### 5. 更多nginx-rtmp配置
   更多的nginx配置可以查看github上的<a href="https://github.com/arut/nginx-rtmp-module">nginx-rtmp-module文档</a>
   


   