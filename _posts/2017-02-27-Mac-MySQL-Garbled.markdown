---
layout:     post
title:      "完美解决Mac下Navicat Premium连接MySQL5.7中文乱码！！！"
subtitle:   "只需要加几个配置就可以了。"
date:       2017-02-27
author:     "Binux"
header-img: "img/in-post/Mac-MySQL/blog.png"
catalog: true
tags:
    - Mac
    - MySQL
---

> “记录下 毕竟也是当初困扰自己好久的问题！”


## 前言

学生自己买不起Mac只能自己装的黑苹果 自己不想研究 还找人花钱装的 (其实自己装也可以 就是有点麻烦) 初用Mac OS 感觉 系统好炫酷 简直就是为我而生的

没用多久就感觉 以后绝对不会用 win10了 下部电脑绝对自己赚钱买Mac 自己慢慢摸索把一些自己常用的软件都装好了 也包括一些全新的软件 **Alfred、1Password、PopClip等** 有时间可以记录下我使用的一些 软件
绝对用过了就回不去的 

好了 回到正题 在使用Navicat Premium 连接MySQL时出现了乱码 查了些资料 

我总结出来的只需要改2处 **有一处还是自己多此一举** 出现的！

---

## 正文

首先我们来改MySQL的字符集

登录MySQL SHOW VARIABLES LIKE "character%";
```
mysql> SHOW VARIABLES LIKE "character%";
+--------------------------+-----------------------------------------------------------+
| Variable_name            | Value                                                     |
+--------------------------+-----------------------------------------------------------+
| character_set_client     | utf8                                                      |
| character_set_connection | utf8                                                      |
| character_set_database   | latin1                                                    |
| character_set_filesystem | binary                                                    |
| character_set_results    | utf8                                                      |
| character_set_server     | latin1                                                    |
| character_set_system     | utf8                                                      |
| character_sets_dir       | /usr/local/mysql-5.7.17-macos10.12-x86_64/share/charsets/ |
+--------------------------+-----------------------------------------------------------+
8 rows in set (0.00 sec)
```
character_set_database和character_set_server的默认字符集还是latin1。

需要改成
```
mysql> SHOW VARIABLES LIKE "character%";
+--------------------------+-----------------------------------------------------------+
| Variable_name            | Value                                                     |
+--------------------------+-----------------------------------------------------------+
| character_set_client     | utf8                                                      |
| character_set_connection | utf8                                                      |
| character_set_database   | utf8                                                      |
| character_set_filesystem | binary                                                    |
| character_set_results    | utf8                                                      |
| character_set_server     | utf8                                                      |
| character_set_system     | utf8                                                      |
| character_sets_dir       | /usr/local/mysql-5.7.17-macos10.12-x86_64/share/charsets/ |
+--------------------------+-----------------------------------------------------------+
8 rows in set (0.00 sec)
```

#### 解决办法

停止MySQL服务
<img src="/img/in-post/Mac-MySQL/img1.png" />
<small class="img-hint">MySQL设置</small>

打开终端 执行 sudo vim /etc/my.cnf 输入密码 新建一个MySQL配置文件 因为5.7 安装的时候默认没有配置文件

加入2条配置
```
[client]
default-character-set=utf8
[mysqld]
character-set-server=utf8
```
重新启动MySQL服务

打开Navicat Premium 连接
<img src="/img/in-post/Mac-MySQL/img2.png" />
<small class="img-hint">Navicat Premium连接</small>

发现还是乱码
<img class="shadow" src="/img/in-post/Mac-MySQL/img3.png" />
<small class="img-hint">中文乱码</small>

其实到这边 MySQL乱码就配置完成了 问题就出在 新建连接的时候Encoding这边选择了utf-8

把它改成Auto 问题解决
<img class="shadow" src="/img/in-post/Mac-MySQL/img4.png" />
<small class="img-hint">问题解决</small>

---

## 总结

出现这个问题不怪别人 都是自己手残 干嘛自己去改连接属性里面的Encoding 其实默认就是Auto 其实这个问题困扰我挺久的 

不过在改过MySQL配置后 使用MyBatis 也就是JDBC操作数据库是没问题的 所以也没在意

最后怎么看自己配置的有没有问题呢！

Navicat Premium右键Edit DataBase 如果是下图 就绝对没问题了
<img src="/img/in-post/Mac-MySQL/img5.png" />
<small class="img-hint">Edit DataBase</small>

---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
