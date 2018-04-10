---
layout: post
title: "ngnix 配置优化"
description: ""
category: cluster
tags: []
---

### main 优化

**1.调整参数隐藏nginx软件版本号**

```
http {
    ……
    server_tokens off;
    ……
}

```
---

**2.更改nginx服务的默认用户**

```
grep "#user" /opt/application/ngnix/nginx/conf/nginx.conf.default
#user nobody
```

    (1).新建用户

```
useradd nginx -s /sbin/nologin -M
```
    (2).修改nginx的配置文件

```
user nginx nginx
```

ps :或者在编译nginx的时候就指定用户和所属组

```
./configure --user=nginx --group=nginx  ……

```

**3.nginx 配置参数优化**

    搭建服务器时，worker进程数最开始的设置等于cpu的核数，高并发场合下可以考虑将进程数提高至cpu * 2，也可以再大点，根据具体的硬件和业务来调整。

```
#查看cpu的核心数
    grep processor /proc/cpuinfo | wc -l

#编辑nginx的配置文件
    vim ngnix.conf
    worker_processes  2;
```


**4.绑定不同的nginx进程到不同的cpu上。**

```
#编辑nginx的配置文件
    vim ngnix.conf
    worker_processes  4;
#0001 0010 0100 1000是掩码，分别代表1,2,3,4核cpu。
    worker_cpu_affinity 0001 0010 0100 1000; 

```

ps： 配置该项过后，进行top命令查看cpu的负载情况，差别不是很大，nginx本身就会做这方面的优化。

---

### events 优化

**1.nginx事件处理模型选择**

```
根据不同的系统选择不同的事件处理模型，可供选择有“kqueue | rstig | epoll | /dev/poll | select | poll  ”
events {
    use epoll;
}

#select ,poll 是标准的工作模式。kqueue ，epoll 是搞笑工作模式，epoll长用在linux平台，kqueue 常用在BSD系统。solaris系统中常用/dev/poll 

```

**2.单进程允许客户端最大连接数**

```
events {
    #默认是1024，可以修改大点，65535.
    worker_connections 65535;
}

```

ps ： 进程的最大连接数受linux系统的最大文件打开数限制，执行“ulimit -HSn 65535” 修改文件最大文件打开数。

**3.worker 的最大文件打开数**

```
worker_rlimit_nofile 该参数与系统优化执行“ulimit -HSn 65535”,后，保持一致。
events {
    worker_connections 1024;
    worker_rlimit_nofile 65535;
}

```

---

### http配置块优化

**1.优化服务器域名散列表大小**

```
http {
    server_names_hash_max_size 512;  #默认是512，一般设置为cpu 以及缓存的4-5倍。
    server_names_hash_bucket_size 128;
}
```

**2.开启高效文件传输模式**

```
http {
    sendfile on ;
    #防止网络及磁盘I/O阻塞。
    tcp_nopush on;
}
```

**3.nginx连接超时参数设置**

```
http {
    keeplive_timeout 3; #默认60s
    tcp_nodelay on;
    client_header_timeout 15s;  #默认60s
    client_body_timeout 15s;    #默认60s
    send_timeout 25s;       #默认60s
}
```

**4.上传文件大小限制**

```
http {
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_max_body_size 8m;
}
```
---

### fastCGI参数配置优化

```
http｛
    fastcgi_connect_timeout 300;  #默认60s
    fastcgi_send_timeout 300;     #默认60s
    fastcgi_read_timeout 300;     #默认60s
    fastcgi_buffer_size 256k;
    fastcgi_buffers 16 256k;
    fastcgi_busy_buffers_size 512k;
    fastcgi_temp_file_write_size 512k;
    fastcgi_intercept_errors on;
    #levels=2:2 设置目录层级，生成256*256个子目录 inactive默认失效时间，max_size 使用最大磁盘空间。
    fastcgi_cache_path /data/ngx_fcgi_cache levels=2:2 keys_zone=ngx_fcgi_cache:512m inactive=1d max_size=40g;
    server {
        listen 80;
        server_name www.dnzhu-blog.com;
        root html/blog;
        location / {
            root      html/blog;
            index index.php index.html index.htm;
        }
        location ~ .*\.(php|php5)?$ {
            fastcgi_pass  127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
            fastcgi_cache ngx_fcgi_cache;
            fastcgi_cache_valid 200 302 1h;   #200,302 应答缓存1小时
            fastcgi_cache_valid 301 1d;  #301 应答缓存1天
            fastcgi_cache_valid any 1m;  #其他应答缓存1分钟
            fastcgi_cache_min_uses 1;        #设置请求几次后，响应被缓存
            fastcgi_cache_use_stale error timeout invalid_header http_500; #在哪些情况下使用过期缓存
            fastcgi_cache_key http://$host$request_uri;  
        }
    }
｝

```
---

### nginx配置gzip 压缩性能优化

**1.为什么要压缩**

    1.提升用户体验，发送的内容少，网页加载的快。
    2.节省带宽成本。

**压缩的对象**

    纯文本压缩比例比较高，最好进行压缩。如： html ,js,css ,xml, shtml等。
    被压缩的内容必须大于1kb，极小的文件压缩后可能会更大。
    图片，视频等文件尽量不压缩，这些文件大多数是经过压缩处理的，压缩大文件可能会消耗大量的cpu ，内存资源。

参数说明：

```
http {
    gzip on;               #开启压缩
    gzip_min_length  1k;   #允许压缩页面的最小字节数
    gzip_buffers     4 16k;   #压缩缓冲区大小
    gzip_http_version 1.1;    #压缩版本
    gzip_comp_level 2;    #压缩比率，1是压缩比例最小，处理速度最快，9是压缩比大，速度最慢
    #支持的压缩类型
    gzip_types       text/plain application/x-javascript text/css application/xml;
    gzip_vary on;     #支持前端的缓存服务器缓存经过gzip压缩的页面。
}
```
---

### nginx 配置expires缓存实现性能优化

    expires功能就是允许通过nginx配置文件控制http的expires和cache-control响应头的内容。告送浏览器是否缓存和缓存多长时间。

**(1).根据文件扩展名进行判断**

```
location ~ .*\.(gif | jpg | jpeg | png | nmp | swf)$ {
    expires 365d;
}  
```

**(2).缓存某个特定的文件**

```
location ~(robots.txt) {
    expires 7d;
    break;
}
```

**设置nginx缓存的两面性：涉及到缓存更新机制的问题**

    1）.对于经常变动或更新的数据，缩短缓存周期。
    2）.网站改版或更新时，可以在服务器将缓存对象改名。同步参数控制不同的版本。

---

### nginx 日志相关优化

**(1).配置日志切割脚本**

```
vim cut_nginx_log.sh

#!/bin/bash
cd /opt/application/nginx/nginx/logs && \ /bin/mv blog_access.log blog_access_$(date +%F -d -1day).log
/opt/application/ngnix/nginx/sbin/nginx -s reload
```

```
crontab -e
00 00 * * * /bin/sh /usr/local/cut_nginx_log.sh >/dev/null 2>&1
```

**(2).不记录不需要的访问日志**

如果日志写入太频繁，会占用大量的磁盘I/O，从而降低了服务器的性能。

```
location ~ .*\.(js | css | gif | jpg | jpeg | png | nmp | swf)$ {
    access_log off ;
}
```

**(3).访问日志权限设置**

不需要在日志目录上给nginx用户可读可写权限，会有一定的安全隐患。

```
chown -R root.root /opt/application/nginx/nginx/logs
chmod -R 700 /opt/application/nginx/nginx/logs
```

---

### 限制IP访问

```
location / {
    root html/blog;
    index.index.php index.html index.htm;
    deny 10.20.0.56;
    allow 127.0.0.1/24;
    allow 192.168.0.0/16;
    allow 10.10.0.0/16;
    deny all;
}
```

注意： 
    deny 一定要加一个ip，否则调转到403。
    对于allow的ip段，允许访问的段位从小到大排列。
    24,代表子网掩码：255.255.255.0
    16,代表子网掩码：255.255.0.0
    8,代表子网掩码：255.0.0.0

---

### nginx图片及目录防盗链

```
location ~* ^.+\.(js | css | gif | jpg | jpeg | png | nmp | rar | zip )$ {
    valid_referers none blocked server_names *.dnzhu-blog.com dnzhu-blog.com;
    if ($invalid_referer) {
        rewrite ^/ http://www.dnzhu-blog.com;
    }
    access_log off;
    root html/blog;
    expires 1d;
    break;
}

location /images {
    root html/data/img;
    valid_referers none blocked  *.dnzhu-blog.com dnzhu-blog.com;
    if ($invalid_referer) {
        return 403;
    }
}

```
---

### 优雅的显示错误页面

给一个tmall的demo：

```
error_page 500 501 502 503 504  https://err.tmall.com/error2.html
error_page 400 403 404 405 408 410 411 412 413 414 415  https://err.tmall.com/error1.html

```

### nginx 企业网站集群超级安全设置

    (1).动态web服务器集群 ：目录权限755 ，文件权限644，所用的目录及文件的用户和组都是root。
    (2).静态服务器集群 ： 目录权限755，文件权限644，所用的目录及文件的用户和组都是root。
    上传upload集群 :  禁止解析 .php /.sh 文件，程序目录：f 644 d755  root root ,上传目录： f 644 d755  web web 。

---

### 控制nginx并发连接数量

```
ngx_http_limit_conn_module 这个模块用于限制key的连接数。
语法：limit_conn_zone key zone=name:size;
上下文：http

语法：limit_conn num
上下文：http server location
```

    1.限制单ip并发连接数

```
http {
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    ……
    server {
    ……
    location / {
        ……
        limit_conn addr 1;  #限制单ip的并发连接为1
        }
    }
}

```

    2.限制虚拟主机总连接数

```
http {
    limit_conn_zone $binary_remote_addr zone=addr:10m;
    limit_conn_zone $server_name zone=perserver:10m;
    ……
    server {
    ……
    location / {
        ……
        #limit_conn addr 1;  #限制单ip的并发连接为1
        limit_conn perserver 2;设置虚拟主机的连接数为2
        }
    }
}
```

---

### 控制客户端请求nginx的速率

    ngx_http_limit_req_module 模块用于限制每个ip访问key的请求速率。 

    语法：limit_req_zone key zone=name:size rate=rate;
    
    上下文：http

    语法：limit_req zone=name [burst=number] [nodelay]
    上下文：http server location

```
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    #以客户端ip为key，one是内存区域名，分配10m空间，速率为1秒1次请求。
    ……
    server {
    ……
    location / {
        ……
        limit_req one burst=5; #队列值为5，可以5个请求队列等待。
        }
    }
}
```


