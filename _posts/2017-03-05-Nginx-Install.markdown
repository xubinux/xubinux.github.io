---
layout:     post
title:      "Xbin-Store(分布式商城)项目所用Linux服务系列 Nginx安装(四)"
subtitle:   "————Nginx安装"
date:       2017-03-04
author:     "Binux"
header-img: "img/in-post/Linux-Nginx/blog.png"
catalog: true
tags:
    - XBin-Store
    - Nginx
    - Linux
---

> “这篇文章将介绍如何安装Nginx。”

## 系列
* [Xbin-Store(分布式商城)项目所用Linux服务系列 MySQL安装(一)](http://binux.cn/2017/03/01/Linux-MySQL-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Redis集群安装(二)](http://binux.cn/2017/03/03/Redis-Cluster-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Zookeeper集群安装(三)](http://binux.cn/2017/03/04/Zookeeper-Cluster-Install/)
* **[Xbin-Store(分布式商城)项目所用Linux服务系列 Nginx安装(四)](http://binux.cn/2017/03/04/Nginx-Install/)**
* [Xbin-Store(分布式商城)项目所用Linux服务系列 FastDFS安装(五)](http://binux.cn/2017/03/05/FastDFS-Install/)
* [Xbin-Store(分布式商城)项目所依赖的Linux服务器 安装系列(六)](http://binux.cn/2017/03/06/Solr-Cluster-Install/)


## 前言
本篇Nginx安装基于CentOS6.5 将会介绍Nginx的安装

Nginx的负载均衡和高可用 这篇将不会涉及

> 机器IP 192.168.1.1

---

## 正文

### wget下载:
http://nginx.org/download/nginx-1.4.2.tar.gz

### 安装： tar -zxvf nginx-1.6.2.tar.gz

### 下载需要的依赖库文件：
```bash
yum install pcre
yum install pcre-devel
yum install zlib
yum install zlib-devel
```
###  进行configure配置：
cd nginx-1.6.2 && ./configure --prefix=/usr/local/nginx  查看是否报错
### 编译安装 make && make install

### 启动Nginx：

### 启动命令：
/usr/local/nginx/sbin/nginx -s start 关闭（stop）重启（reload）

#### 成功：
查看是否启动（netstat -ano | grep 80）
#### 失败：
可能为80端口被占用等。
### 访问页面：
浏览器访问地址：http://192.168.1.1:80 （看到欢迎页面即可）

## 配置文件说明
```bash
#user  nobody;

#开启进程数 <=CPU数
worker_processes  1;

#错误日志保存位置
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#进程号保存文件
#pid        logs/nginx.pid;

#每个进程最大连接数（最大连接=连接数x进程数）每个worker允许同时产生多少个链接，默认1024
events {
    worker_connections  1024;
}


http {
	#文件扩展名与文件类型映射表
    include       mime.types;
	#默认文件类型
    default_type  application/octet-stream;

	#日志文件输出格式 这个位置相于全局设置
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

	#请求日志保存位置
    #access_log  logs/access.log  main;

	#打开发送文件
    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
	#连接超时时间
    keepalive_timeout  65;

	#打开gzip压缩
    #gzip  on;

	#设定请求缓冲
	#client_header_buffer_size 1k;
	#large_client_header_buffers 4 4k;

	#设定负载均衡的服务器列表
	#upstream myproject {
		#weigth参数表示权值，权值越高被分配到的几率越大
		#max_fails 当有#max_fails个请求失败，就表示后端的服务器不可用，默认为1，将其设置为0可以关闭检查
		#fail_timeout 在以后的#fail_timeout时间内nginx不会再把请求发往已检查出标记为不可用的服务器
	#}

    #webapp
    #upstream myapp {
  	# server 192.168.1.1:8080 weight=1 max_fails=2 fail_timeout=30s;
	# server 192.168.1.1:8080 weight=1 max_fails=2 fail_timeout=30s;
    #}

	#配置虚拟主机，基于域名、ip和端口
    server {
		#监听端口
        listen       80;
		#监听域名
        server_name  localhost;

        #charset koi8-r;

		#nginx访问日志放在logs/host.access.log下，并且使用main格式（还可以自定义格式）
        #access_log  logs/host.access.log  main;

		#返回的相应文件地址
        location / {
            #设置客户端真实ip地址
            #proxy_set_header X-real-ip $remote_addr;
			#负载均衡反向代理
			#proxy_pass http://myapp;

			#返回根路径地址（相对路径:相对于/usr/local/nginx/）
            root   html;
			#默认访问文件
            index  index.html index.htm;
        }

		#配置反向代理tomcat服务器：拦截.jsp结尾的请求转向到tomcat
        #location ~ \.jsp$ {
        #    proxy_pass http://192.168.1.1:8080;
        #}

        #error_page  404              /404.html;
        # redirect server error pages to the static page /50x.html
        #

		#错误页面及其返回地址
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

	#虚拟主机配置：
	server {
		listen 1234;
		server_name bhz.com;
		location / {
		#正则表达式匹配uri方式：在/usr/local/nginx/bhz.com下 建立一个test123.html 然后使用正则匹配
		#location ~ test {
			## 重写语法：if return （条件 = ~ ~*）
			#if ($remote_addr = 192.168.1.200) {
			#       return 401;
			#}

			#if ($http_user_agent ~* firefox) {
			#	   rewrite ^.*$ /firefox.html;
			#	   break;
			#}

			root bhz.com;
			index index.html;
		}

		#location /goods {
		#		rewrite "goods-(\d{1,5})\.html" /goods-ctrl.html;
		#		root bhz.com;
		#		index index.html;
		#}

		#配置访问日志
		access_log logs/bhz.com.access.log main;
	}



    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

---

## 总结

这篇介绍了Nginx的安装 非常简单 下面会写篇介绍Nginx负载均衡和高可用的博客

---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
