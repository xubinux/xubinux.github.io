---
layout:     post
title:      "解决Java程序连接不上MySQL 但是Navicat可以连接的问题!"
subtitle:   "包括连接不上本地虚拟机中MySQL"
date:       2017-03-02
author:     "Binux"
header-img: "img/in-post/MySQL-ERROR/blog.png"
catalog: true
tags:
    - MySQL
    - Bug
---

> “这篇文章将讲一个困扰了我3天的问题 Java连接不上MySQL”


## 前言

上一篇讲的Mac上MySQL就突然没用了 但是Navicat、IDEA都可以连接上MySQL 使用Java就是连接不上 **重装也没有用**


#### 具体情况是这样的

* 第一天晚上在写订单模块 代码写完了 准备测试时 突然整个控制台狂打印错误 具体的错误就是下面的错误 当时就想着反正就order模块报错 明天再改吧 就回宿舍了

* 第二天早上过来 我启动其他服务 发现全部都报错 Google了一上午 重装MySQL 依然没解决 下午不甘的去上课

* 第二天晚上继续找错 试遍了各种情况 打算在虚拟机装MySQL 直接用虚拟机的 但是 依然报错 但是我发现我把网线一拔就可以正常使用 哎 谁能体会我现在的心情 再次不甘的回宿舍了

* 第三天早上 带着万分的无奈 继续找错 8点找到11点半 还是找不到啊

* 第三天下午我终于找到了 还是得感谢Google 感谢www.stackoverflow.com上的@BalusC

#### 下面我会讲一下这个问题的原因 不限于我出现的这个问题 帮大家分析下这个问题 希望大家不要出现我这种问题了 为什么我老是碰到这种蛋疼问题 **浪费时间还让人心烦** (3天时间干什么不好啊)

---

## 正文
#### @BalusC解答截图 再次感谢
<img class="shadow" src="/img/in-post/MySQL-ERROR/img1.png" />
<small class="img-hint">@BalusC解答截图</small>

#### 具体报错--MySQL服务没启动同样报此错
```
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:57)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:526)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:404)
	at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:981)
	at com.mysql.jdbc.MysqlIO.readPacket(MysqlIO.java:628)
	at com.mysql.jdbc.MysqlIO.doHandshake(MysqlIO.java:1014)
	at com.mysql.jdbc.ConnectionImpl.coreConnect(ConnectionImpl.java:2255)
	at com.mysql.jdbc.ConnectionImpl.connectOneTryOnly(ConnectionImpl.java:2286)
	at com.mysql.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:2085)
	at com.mysql.jdbc.ConnectionImpl.<init>(ConnectionImpl.java:795)
	at com.mysql.jdbc.JDBC4Connection.<init>(JDBC4Connection.java:44)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:57)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:526)
	at com.mysql.jdbc.Util.handleNewInstance(Util.java:404)
	at com.mysql.jdbc.ConnectionImpl.getInstance(ConnectionImpl.java:400)
	at com.mysql.jdbc.NonRegisteringDriver.connect(NonRegisteringDriver.java:327)
	at org.mybatis.generator.internal.db.ConnectionFactory.getConnection(ConnectionFactory.java:68)
	at org.mybatis.generator.config.Context.getConnection(Context.java:526)
	at org.mybatis.generator.config.Context.introspectTables(Context.java:436)
	at org.mybatis.generator.api.MyBatisGenerator.generate(MyBatisGenerator.java:222)
	at org.mybatis.generator.api.MyBatisGenerator.generate(MyBatisGenerator.java:133)
	at GeneratorSqlmap.generator(GeneratorSqlmap.java:27)
	at GeneratorSqlmap.main(GeneratorSqlmap.java:33)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:147)
Caused by: java.io.EOFException: Can not read response from server. Expected to read 4 bytes, read 0 bytes before connection was unexpectedly lost.
	at com.mysql.jdbc.MysqlIO.readFully(MysqlIO.java:2957)
	at com.mysql.jdbc.MysqlIO.readPacket(MysqlIO.java:560)
	... 25 more
```

#### 分析

##### 报错问题：
The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.

最后一个发送成功的数据包在0秒前 没有收到任何来自服务器的数据包。

简单来说就是连不上MySQL服务器了
##### 检查MySQL连接是否有错
这时候你就得检查下你的msyql连接url有没有写错了
```
driverClass="**com.mysql.jdbc.Driver**"
connectionURL="jdbc:mysql://**IP**:**3306**/**库名**"
userId="root"
password="**密码**"
```

> ip 端口 库名 仔细检查下

如果到这还没有解决问题 那么继续往下看

#### 检查Hosts文件
查看jdbc中的IP是否在hosts文件中有解析

#### 检查MySQL配置文件
查看是否绑定ip地址

如果有这行 注释bind-address=xxx.xxx.xxx.xxx
#### 检查MySQL运行在哪个端口
```
mysql> show global variables like 'port';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| port          | 3306  |
+---------------+-------+
1 row in set (0.10 sec)
```
#### 检查MySQL服务是否真正启动
telnet **Ip** 3306
#### 检查MySQL是否不接受远程连接
```
SELECT USER,HOST FROM user;
//正确
+-----------+-----------+
| USER      | HOST      |
+-----------+-----------+
| root      | %         |
| mysql.sys | localhost |
| root      | localhost |
+-----------+-----------+
```
如果没有授权的话

mysql> grant all privileges on *.* to root@'%' identified by '**我是密码**';
#### 检查MySQL是否已经用完了所有连接
重启MySQL 出现运行程序 如果不报错就是此问题

#### 检查有没有什么东西在阻止Java程序和MySQL连接

比如关闭**防火墙** 或者设置防火墙规则

比如有没有开什么**代理**

比如拔网线 再连接MySQL试试

---

## 总结
<img class="shadow" src="/img/in-post/MySQL-ERROR/img2.png" />
<small class="img-hint">Shadowsocks</small>

我是栽在最后一条Proxy 想到了Firewall 却忘记他了 程序员翻个墙看看外面的世界 这是一件很正常的事 个人感觉Google甩Baidu一个地球到月球的距离 我就是由于开着小飞机(Shadowsocks) 嫌网速卡 切换到全局模式 然后就让我郁闷了3天。

改成自动模式 解决问题..

---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
