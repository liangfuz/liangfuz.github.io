---
layout: post
title: Centos yum 搭建与配置Nginx
category: 环境搭建
tags: nginx
keywords: nginx,linux
---

Nginx 是一个安装非常的简单、配置文件非常简洁（还能够支持perl语法）、Bug非常少的高性能HTTP和反向代理服务器，yum安装nginx也非常简单  

## yum 安装 Nginx
### 第一步先获取安装包
`rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm`

### 第二步安装并启动nignx
```
yum install nginx
service nginx start
Starting nginx:              [  OK  ]
```
### 第三步进入浏览器，输入IP或者域名测试
如果看到以下输出则说明安装完成
```
Welcome to nginx!
If you see this page, the nginx web server is successfully installed and working. Further configuration is required.
For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.
Thank you for using nginx.
```

## Nginx 配置
nginx的配置文件是目录下面的nginx.conf  
一份很简单的nginx配置[nginx.conf](http://github-blog.oss-cn-shenzhen.aliyuncs.com/conf/nginx.conf)

### 多服务器负载均衡配置
在server外添加一个upstream，而直接在proxy_pass里面直接用http://+upstream的名称来使用。
upstream中的server元素必须要注意，不能加http://，但proxy_pass中必须加。
如果第一台服务器挂了，就会自动转发到第二台，同时weight决定了哪台服务器更有可能被访问到。
max_fails=2 fail_timeout=120s; 表示server如果在120s内发生2次失败（超时或者拒绝连接）则将该server剔除出去，不再向其分发请求，120秒后再恢复服务。
```
upstream backend {  
    server localhost:8080 #weight=1 max_fails=2 fail_timeout=120s;  
    server localhost:9999 #weight=5 max_fails=2 fail_timeout=120s;  
}  
  
server{  
        location / {  
           proxy_pass http://backend;  
        }  
        #......其他省略  
}  
```

### location匹配命令
```
~      #波浪线表示执行一个正则匹配，区分大小写
~*    #表示执行一个正则匹配，不区分大小写
^~    #^~表示普通字符匹配，如果该选项匹配，只匹配该选项，不匹配别的选项，一般用来匹配目录
=      #进行普通字符精确匹配
@     #"@" 定义一个命名的 location，使用在内部定向时，例如 error_page, try_files
```
### location 匹配的优先级(与location在配置文件中的顺序无关)
```
    1.= 精确匹配会第一个被处理。如果发现精确匹配，nginx停止搜索其他匹配。
    2.普通字符匹配，正则表达式规则和长的块规则将被优先和查询匹配，也就是说如果该项匹配还需去看有没有正则表达式匹配和更长的匹配。
    3.^~ 则只匹配该规则，nginx停止搜索其他匹配，否则nginx会继续处理其他location指令。
    4.最后匹配理带有"~"和"~*"的指令，如果找到相应的匹配，则nginx停止搜索其他匹配；当没有正则表达式或者没有正则表达式被匹配的情况下，那么匹配程度最高的逐字匹配指令会被使用。
```
### 例如
```
    location  = / {
      # 只匹配"/".
      [ configuration A ] 
    }
    location  / {
      # 匹配任何请求，因为所有请求都是以"/"开始
      # 但是更长字符匹配或者正则表达式匹配会优先匹配
      [ configuration B ] 
    }
    location ^~ /images/ {
      # 匹配任何以 /images/ 开始的请求，并停止匹配 其它location
      [ configuration C ] 
    }
    location ~* .(gif|jpg|jpeg)$ {
      # 匹配以 gif, jpg, or jpeg结尾的请求. 
      # 但是所有 /images/ 目录的请求将由 [Configuration C]处理.   
      [ configuration D ] 
    }
```
## nginx常用的超时配置说明

### client_header_timeout
语法 client_header_timeout time
默认值 60s
上下文 http server
说明 指定等待client发送一个请求头的超时时间（例如：GET / HTTP/1.1）.仅当在一次read中，没有收到请求头，才会算成超时。如果在超时时间内，client没发送任何东西，nginx返回HTTP状态码408(“Request timed out”)

### client_body_timeout 
语法 client_body_timeout time
默认值 60s
上下文 http server location
说明 该指令设置请求体（request body）的读超时时间。仅当在一次readstep中，没有得到请求体，就会设为超时。超时后，nginx返回HTTP状态码408(“Request timed out”)

### keepalive_timeout 
语法 keepalive_timeout timeout [ header_timeout ]
默认值 75s
上下文 http server location
说明 第一个参数指定了与client的keep-alive连接超时时间。服务器将会在这个时间后关闭连接。可选的第二个参数指定了在响应头Keep-Alive: timeout=time中的time值。这个头能够让一些浏览器主动关闭连接，这样服务器就不必要去关闭连接了。没有这个参数，nginx不会发送Keep-Alive响应头（尽管并不是由这个头来决定连接是否“keep-alive”）
两个参数的值可并不相同
注意不同浏览器怎么处理“keep-alive”头
MSIE和Opera忽略掉"Keep-Alive: timeout=<N>" header.
MSIE保持连接大约60-65秒，然后发送TCP RST
Opera永久保持长连接
Mozilla keeps the connection alive for N plus about 1-10 seconds.
Konqueror保持长连接N秒

### lingering_timeout
语法 lingering_timeout time
默认值 5s
上下文 http server location
说明 lingering_close生效后，在关闭连接前，会检测是否有用户发送的数据到达服务器，如果超过lingering_timeout时间后还没有数据可读，就直接关闭连接；否则，必须在读取完连接缓冲区上的数据并丢弃掉后才会关闭连接。

### resolver_timeout
语法 resolver_timeout time 
默认值 30s
上下文 http server location
说明 该指令设置DNS解析超时时间

### proxy_connect_timeout
语法 proxy_connect_timeout time 
默认值 60s
上下文 http server location
说明 该指令设置与upstream server的连接超时时间，有必要记住，这个超时不能超过75秒。
这个不是等待后端返回页面的时间，那是由proxy_read_timeout声明的。如果你的upstream服务器起来了，但是hanging住了（例如，没有足够的线程处理请求，所以把你的请求放到请求池里稍后处理），那么这个声明是没有用的，由于与upstream服务器的连接已经建立了。

### proxy_read_timeout
语法 proxy_read_timeout time 
默认值 60s
上下文 http server location
说明 该指令设置与代理服务器的读超时时间。它决定了nginx会等待多长时间来获得请求的响应。这个时间不是获得整个response的时间，而是两次reading操作的时间。

### proxy_send_timeout
语法 proxy_send_timeout time 
默认值 60s
上下文 http server location
说明 这个指定设置了发送请求给upstream服务器的超时时间。超时设置不是为了整个发送期间，而是在两次write操作期间。如果超时后，upstream没有收到新的数据，nginx会关闭连接

### proxy_upstream_fail_timeout（fail_timeout）
语法 server address [fail_timeout=30s]
默认值 10s
上下文 upstream
说明 Upstream模块下 server指令的参数，设置了某一个upstream后端失败了指定次数（max_fails）后，该后端不可操作的时间，默认为10秒

## Nginx 的几个常用指令
```
nginx -t                   # 检查配置文件是否有错
nginx -s reload            # 重新载入配置文件
nginx -s reopen            # 重启 Nginx
nginx -s stop              # 停止 Nginx
```
