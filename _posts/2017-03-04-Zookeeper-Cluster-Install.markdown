---
layout:     post
title:      "Xbin-Store(分布式商城)项目所依赖的Linux服务器 安装系列(三)"
subtitle:   "————Zookeeper集群安装"
date:       2017-03-04
author:     "Binux"
header-img: "img/in-post/Linux-Zookeeper/blog.png"
catalog: true
tags:
    - XBin-Store
    - Zookeeper
    - Linux
---

> “这篇文章将介绍如何安装Zookeeper集群”


## 前言
### Zookeeper集群方案

主机IP		|	消息端口		|	    通信端口		|节点目录/usr/local/下|
192.168.1.1	|	2181		|		2888:3888	|		zookeeper   |
192.168.1.2	|	2181		|		2888:3888	|	    zookeeper   |
192.168.1.3	|	2181		|		2888:3888	|		zookeeper   |


---

## 正文

### 安装
> 以下操作3台机器同时操作

#### 新建用户
useradd zookeeper

passwd zookeeper

设置密码

#### 下载

官网下载zookeeper-3.4.6：http://apache.fayea.com/zookeeper/

Linux wget http://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz

#### 解压
tar -zxvf zookeeper-3.4.6.tar.gz —C /usr/local

#### 改名
cd /usr/local

mv zookeeper-3.4.6 zookeeper

#### 建立以下文件夹

cd /usr/local/zookeeper

mkdir data

mkdir logs

#### 修改配置文件
将/conf目录下的zoo_sample.cfg文件拷贝一份， 命名为为zoo.cfg

```bash
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
dataDir=/usr/local/zookeeper/data
dataLogDir=/usr/local/zookeeper/logs
# the port at which the clients will connect
clientPort=2181
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
# 2888 端口号是 zookeeper 服务之间通信的端口。
# 3888 是 zookeeper 与其他应用程序通信的端口。
server.0=192.168.1.1:2888:3888
server.1=192.168.1.2:2888:3888
server.2=192.168.1.3:2888:3888
```

#### 解释配置文件
```
initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不
是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到
Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。
当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没
有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是
5*2000=10 秒。

syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时
间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4
秒。

server.A=B:C:D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务
器的 IP 地址或/etc/hosts 文件中映射了 IP 的主机名；C 表示的是这个服务器与
集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务
器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是
用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是
一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同
的端口号。

```

#### 新建myid文件
cd /usr/local/zookeeper/data

* 192.168.1.1机器:  echo 1 >> myid
* 192.168.1.2机器:  echo 2 >> myid
* 192.168.1.3机器:  echo 3 >> myid

#### 编辑.bash_profile文件
添加
```bash
export ZOOKEEPER_HOME=/usr/local/zookeeper
# zookeeper env
export PATH=$ZOOKEEPER_HOME/bin:$PATH
```
使配置文件生效
$ source /home/zookeeper/.bash_profile

#### 添加防火墙规则

在防火墙中打开要用到的端口2181、2888、3888

##### 切换到 root 用户权限，执行以下命令：
```
chkconfig iptables on
service iptables start
vim /etc/sysconfig/iptables
```
##### 增加以下 3 行：
```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2181 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 2888 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3888 -j ACCEPT
```

##### 重启防火墙：

service iptables restart

### zk操作

#### 启动zk
zkServer.sh start

##### 查看有无启动成功
jps
```
1456 QuorumPeerMain
1475 Jps
```
> 启动成功

#### 查看状态
zkServer.shstatus

#### 查看日志文件
$ tail -500f /usr/local/zookeeper/logs/zookeeper.out

#### 停止zookeeper进程：
zkServer.sh stop

#### 配置zookeeper开机使用xubin用户启动：

编辑/etc/rc.local 文件，加入：

su - zookeeper -c '/usr/local/zookeeper/bin/zkServer.sh start'

### zookeeper查看插件
#### eclipse
* Help -> Install New Software
* 添加 url http://www.massedynamic.org/eclipse/updates/
* 选择zookeeper插件安装
* Eclipse 菜单打开Window -> Show View -> Other… -> ZooKeeper 3.2.2
* 输入zookeeper的地址 连接

#### idea插件
直接插件库中搜索zookeeper
> 不过我不推荐使用
> 插件开发成那样还上线...(不过是免费的 想用就用吧)

#### Java开发的客户端
[点我下载](http://download.csdn.net/detail/nihaoadam/9427620)

---

## 总结
安装集群的时候一定要把防火墙配置好 不然就会出现各种问题 或者直接把防火墙关了吧 学习技术的时候可以不用考虑防火墙

后面会写一篇关于使用ZkClient、Curator操作Zookeeper的博客

---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
