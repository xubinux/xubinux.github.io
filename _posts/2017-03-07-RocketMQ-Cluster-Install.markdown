---
layout:     post
title:      "Xbin-Store(分布式商城)项目所用Linux服务系列 RocketMQ集群安装(七)"
subtitle:   "————RocketMQ集群安装"
date:       2017-03-07
author:     "Binux"
header-img: "img/in-post/Linux-RocketMQ/blog.png"
catalog: true
tags:
    - XBin-Store
    - RocketMQ
    - Linux
---

> “这篇文章将介绍如何安装Solr集群,如何对Solr集群集群进行操作,以及使用Java客户端进行操作!”

## 系列
* [Xbin-Store(分布式商城)项目所用Linux服务系列 MySQL安装(一)](http://binux.cn/2017/03/01/Linux-MySQL-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Redis集群安装(二)](http://binux.cn/2017/03/03/Redis-Cluster-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Zookeeper集群安装(三)](http://binux.cn/2017/03/04/Zookeeper-Cluster-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Nginx安装(四)](http://binux.cn/2017/03/04/Nginx-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 FastDFS安装(五)](http://binux.cn/2017/03/05/FastDFS-Install/)
* [Xbin-Store(分布式商城)项目所依赖的Linux服务系列 Solr集群安装(六)](http://binux.cn/2017/03/06/Solr-Cluster-Install/)
* **[Xbin-Store(分布式商城)项目所依赖的Linux服务系列 RocketMQ集群安装(七)](http://binux.cn/2017/03/07/RocketMQ-Cluster-Install/)**

## 前言
### 所用虚拟机
CentOS 6.5 * 4

RocketMQ 版本3.2.6

模式: 多Master多Slave模式，异步复制

### IP

序号|IP|用户名|密码|角色|模式|
1|192.168.1.1|root|*****|nameServer1,broker-a|Master1|
2|192.168.1.2|root|*****|nameServer2,broker-b|Master2|
3|192.168.1.3|root|*****|nameServer3,broker-a-s|Slave1|
4|192.168.1.4|root|*****|nameServer4,broker-b-s|Slave2|


### 下载软件
* [alibaba-rocketmq-3.2.6.tar.gz](http://download.csdn.net/detail/cynicismsrs/9773419)
* [rocketmq-console](https://github.com/JoeyFan/rocketmq-console)

---

## 正文

### 安装

#### 安装JDK 略

#### 解压 (4台同时)

```bash
tar -zxvf alibaba-rocketmq-3.2.6.tar.gz -C /usr/local
cd /usr/local
ln -s alibaba-rocketmq rocketmq
```

#### 创建存储路径

```bash
mkdir /usr/local/rocketmq/store
mkdir /usr/local/rocketmq/store/commitlog
mkdir /usr/local/rocketmq/store/consumequeue
mkdir /usr/local/rocketmq/store/index
```

#### 配置RocketMQ配置文件

```bash
vim /usr/local/rocketmq/conf/2m-2s-async/broker-a.properties
vim /usr/local/rocketmq/conf/2m-2s-async/broker-b.properties
vim /usr/local/rocketmq/conf/2m-2s-async/broker-a-s.properties
vim /usr/local/rocketmq/conf/2m-2s-async/broker-b-s.properties
```

##### 配置文件

```bash
#所属集群名字 （注意不要有空格）
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样 192.168.1.1/192.168.1.3 填broker-a 192.168.1.2/192.168.1.4 填broker-b
brokerName=(broker-a|broker-b)
#0 表示 Master，>0 表示 Slave  192.168.1.1/192.168.1.2 填0 192.168.1.3/192.168.1.4 填1
brokerId=0
#本机IP 默认识别 多块网卡会导致识别错误
brokerIP1=本机IP
#nameServer地址，分号分割
namesrvAddr=192.168.1.1:9876;192.168.1.2:9876;192.168.1.3:9876;192.168.1.4:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88

#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536

#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000

#Broker 的角色  192.168.1.1/192.168.1.2 填ASYNC_MASTER 192.168.1.3/192.168.1.4 填SLAVE
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER

#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
#filterNums
#filterServerNums=1
```

#### 修改日志配置文件

```bash
mkdir -p /usr/local/rocketmq/logs
cd /usr/local/rocketmq/conf && sed -i 's#${user.home}#/usr/local/rocketmq#g' *.xml
```

#### 修改启动脚本参数（测试时虚拟机内存不够修改）

```bash
vim /usr/local/rocketmq/bin/runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m - XX:PermSize=128m -XX:MaxPermSize=320m"

vim /usr/local/rocketmq/bin/runserver.sh
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m - XX:PermSize=128m -XX:MaxPermSize=320m"
```

#### 启动NameServer

nohup sh /usr/local/rocketmq/bin/mqnamesrv &

##### 查看是否启动成功
tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log

#### 启动BrokerServer 多Master多Slave模式，异步复制

> A -192.168.1.1|B-192.168.1.2|C-192.168.1.3|B-192.168.1.4

启动命令:
```bash
A:
cd /usr/local/rocketmq/bin
nohup sh /usr/local/rocketmq/bin/mqbroker -c /usr/local/rocketmq/conf/2m-2s-async/broker-a.properties >/dev/null 2>&1 &
//查看是否启动成功
netstat -ntlp
jps
// 查看日志
tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log
tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log

B:
cd /usr/local/rocketmq/bin
nohup sh /usr/local/rocketmq/bin/mqbroker -c /usr/local/rocketmq/conf/2m-2s-async/broker-b.properties >/dev/null 2>&1 &
//查看是否启动成功
netstat -ntlp
jps
// 查看日志
tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log
tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log

C:
cd /usr/local/rocketmq/bin
nohup sh /usr/local/rocketmq/bin/mqbroker -c /usr/local/rocketmq/conf/2m-2s-async/broker-a-s.properties >/dev/null 2>&1 &
//查看是否启动成功
netstat -ntlp
jps
// 查看日志
tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log
tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log

D:
cd /usr/local/rocketmq/bin
nohup sh /usr/local/rocketmq/bin/mqbroker -c /usr/local/rocketmq/conf/2m-2s-async/broker-b-s.properties >/dev/null 2>&1 &
//查看是否启动成功
netstat -ntlp
jps
// 查看日志
tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log
tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log
```

#### 安装Tomcat配置RocketMQ Console

复制rocketmq-console.war到192.168.1.1机器的/usr/local/tomcat/webapps/下

vim /usr/local/tomcat/webapps/rocketmq-console/WEB-INF/classes/config.properties

rocketmq.namesrv.addr=192.168.1.1:9876;192.168.1.2:9876;192.168.1.3:9876;192.168.1.4:9876

##### 启动：
/usr/local/tomcat/bin/startup.sh

#### 防火墙配置：
vim /etc/sysconfig/iptables

```bash
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 9876 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 10911 -j ACCEPT
```
重启防火墙

service iptables restart

---

## 总结
虽然RocketMQ开源的是被阉割版 但是现在项目已经捐献给Apache了 现在已经发布了4.0 相信这款消息中间件以后一定会大放光彩


---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
