---
layout:     post
title:      "Xbin-Store(分布式商城)项目所依赖的Linux服务器 安装系列(一)"
subtitle:   "————MySQL5.7安装"
date:       2017-03-01
author:     "Binux"
header-img: "img/in-post/Linux-MySQL/blog.png"
catalog: true
tags:
    - XBin-Store
    - MySQL
    - Linux
---

> “这篇文章将介绍如何Linux下如何安装MySQL 只适合Linux小白看 大神勿喷”


## 前言

Mac上MySQL就突然没用了 但是Navicat、IDEA都可以连接上MySQL 使用Java就是连接不上 **重装也没有用**

 Google、Baidu了一上午都没解决 不想在浪费时间了 就在Linux中装个MySQL用吧！ (虽然很想把问题找出来)

---

## 正文

### 第一步 安装CentOS 略

### 第二步 下载MySQL
* 1、Linux下载 wget http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
* 2、[右键我迅雷下载](http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz)

### 第三步 解压tar文件
\[root@mysql ~\]# tar -zxvf mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz

### 第四步 改名
\[root@mysql ~\]# mv mysql-5.7.17-linux-glibc2.5-x86_64 /usr/local/mysql

### 第五步 复制修改配置文件
\[root@mysql ~\]# cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf

\[root@mysql ~\]# vim /etc/my.cnf

#### 添加以下配置：(解决中文乱码)
```
[mysql]
default-character-set=utf8

[mysqld]
character_set_server=utf8
```

### 第六步 复制、配置mysql服务
\[root@mysql ~\]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql

#### 第46行做如下修改
```
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
```

### 第七步 启动mysql服务

\[root@mysql ~\]# cd /usr/local/mysql/bin/

\[root@mysql bin\]# ./mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

#### 屏幕打印：
```
2017-01-27T03:14:04.598579Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2017-01-27T03:14:04.598645Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2017-01-27T03:14:04.598648Z 0 [Warning] 'NO_AUTO_CREATE_USER' sql mode was not set.
2017-01-27T03:14:05.002057Z 0 [Warning] InnoDB: New log files created, LSN=45790
2017-01-27T03:14:05.079640Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2017-01-27T03:14:05.214710Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: a8c7be60-e43e-11e6-b470-000c299fe1ef.
2017-01-27T03:14:05.216074Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2017-01-27T03:14:05.221297Z 1 [Note] A temporary password is generated for root@localhost: ME#SiCHkw4zS
```

> ME#SiCHkw4zS 这个是初始化密码

### 第八步 加密mysql数据
\[root@mysql mysql\]# ./bin/mysql_ssl_rsa_setup --datadir=/usr/local/mysql/data

#### 屏幕打印：
```
Generating a 2048 bit RSA private key
....................................................+++
..................+++
writing new private key to 'ca-key.pem'
-----
Generating a 2048 bit RSA private key
......+++
.............+++
writing new private key to 'server-key.pem'
-----
Generating a 2048 bit RSA private key
..........................................................................................................................................+++
.....+++
writing new private key to 'client-key.pem'
-----
```

### 第九步 启动mysql
\[root@mysql mysql\]# ./mysqld_safe --user=mysql &

#### 查看启动是否成功(打印如下 启动成功)

\[root@mysql mysql\]# ps -ef\|grep mysql

#### 屏幕打印：
```
root      2286  1573  0 11:30 pts/1    00:00:00 /bin/sh ./mysqld_safe --user=mysql
mysql     2407  2286  5 11:30 pts/1    00:00:00 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/usr/local/mysql/data/storm3.err --pid-file=/usr/local/mysql/data/mysql.pid
root      2438  1573  0 11:30 pts/1    00:00:00 grep mysql
```

#### 启动 输入初始密码
\[root@mysql mysql\]# ./mysql -uroot -p

### 第十步 重新设置密码、授权
#### 设置密码
mysql> grant all privileges on *.* to root@'%' identified by '**我是密码**';

#### 授权
mysql>grant all privileges on *.* to root@'%' identified by '**我是密码**';

#### 刷新
mysql>flush privileges;
### 第十一步 设置防火墙规则

\[root@mysql mysql\]# vim /etc/sysconfig/iptables

#### 添加以下规则：
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
```
#### 重启防火墙：
\[root@mysql mysql\]# service iptables restart

### 第十二步 开机自启

chkconfig --add mysql

chkconfig mysql on

### 第十三步 配置环境变量
\[root@mysql mysql\]# vim /etc/profile

#### 添加配置
export PATH=$JAVA_HOME/bin:/usr/local/mysql/bin:$PATH

### 第十四步 Linux新建mysql用户
\[root@mysql mysql\]# groupadd mysql

\[root@mysql mysql\]# useradd -r -g mysql mysql

\[root@mysql mysql\]# passwd mysql

\[root@mysql mysql\]#chown -R mysql:mysql /usr/local/mysql/


---

## 总结

MySQL安装还是很简单的。


---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
