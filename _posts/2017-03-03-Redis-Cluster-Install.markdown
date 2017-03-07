---
layout:     post
title:      "Xbin-Store(分布式商城)项目所用Linux服务系列 Redis集群安装(二)"
subtitle:   "————Redis集群安装"
date:       2017-03-03
author:     "Binux"
header-img: "img/in-post/Linux-Redis/blog.png"
catalog: true
tags:
    - XBin-Store
    - Redis
    - Linux
---

> “这篇文章将介绍如何安装Redis集群(6节点 3主3从),并且如何对Redis集群进行操作,以及Jedis进行操作!”

## 系列
* [Xbin-Store(分布式商城)项目所用Linux服务系列 MySQL安装(一)](http://binux.cn/2017/03/01/Linux-MySQL-Install/)
* **[Xbin-Store(分布式商城)项目所用Linux服务系列 Redis集群安装(二)](http://binux.cn/2017/03/03/Redis-Cluster-Install/)**
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Zookeeper集群安装(三)](http://binux.cn/2017/03/04/Zookeeper-Cluster-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Nginx安装(四)](http://binux.cn/2017/03/04/Nginx-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 FastDFS安装(五)](http://binux.cn/2017/03/05/FastDFS-Install/)
* [Xbin-Store(分布式商城)项目所依赖的Linux服务器 Solr集群安装(六)](http://binux.cn/2017/03/06/Solr-Cluster-Install/)
* [Xbin-Store(分布式商城)项目所依赖的Linux服务器 RocketMQ集群安装(七)](http://binux.cn/2017/03/07/RocketMQ-Cluster-Install/)

## 前言
本篇安装的Redis版本为3.0 目前官方最新稳定版3.2

需要安装gcc：yum install gcc-c++

需要安装ruby的环境。

yum -y install ruby ruby-devel rubygems rpm-build

需要使用到官方提供的ruby脚本[redis-3.0.0.gem](https://rubygems.org/downloads/redis-3.0.0.gem)。

gem install redis-3.0.0.gem

> 本机IP为192.168.1.1

---

## 安装

### 下载源码[点我下载]( http://download.redis.io/releases/redis-3.0.0.tar.gz)

### 解压
tar -zxvf redis-3.0.0.tar.gz

### 编译
cd /usr/local/redis-3.0.0

make

### 安装
make PREFIX=/usr/local/redis1 install

### 拷贝配置文件
进入源码目录，里面有一份配置文件 redis.conf，然后将其拷贝到安装路径下

cd /usr/local/redis

mkdir etc

cp redis-3.0.0/redis.conf  /usr/local/redis-cluster/redis1/etc

编辑redis.conf 打开第632行# cluster-enabled yes 的注释

### 复制6个Redis
分别命名为redis1-6

分别修改redis.conf配置文件中的第45行端口号为7001-7006

### 复制redis-trib.rb集群管理工具
cp redis-3.0.0/src/redis-trib.rb  /usr/local/redis-cluster

### 启动(后端启动)
 /usr/local/redis-cluster/redis1/bin/redis-server /usr/local/redis-cluster/redis1/etc/redis.conf
 /usr/local/redis-cluster/redis2/bin/redis-server /usr/local/redis-cluster/redis2/etc/redis.conf
 /usr/local/redis-cluster/redis3/bin/redis-server /usr/local/redis-cluster/redis3/etc/redis.conf
 /usr/local/redis-cluster/redis4/bin/redis-server /usr/local/redis-cluster/redis4/etc/redis.conf
 /usr/local/redis-cluster/redis5/bin/redis-server /usr/local/redis-cluster/redis5/etc/redis.conf
 /usr/local/redis-cluster/redis6/bin/redis-server /usr/local/redis-cluster/redis6/etc/redis.conf

### 创建集群
cd /usr/local/redis-cluster
./redis-trib.rb create --replicas 1 **192.168.1.1**:**7001** **192.168.1.1**:**7002** **192.168.1.1**:**7003** **192.168.1.1**:**7004** **192.168.1.1**:**7005**  **192.168.1.1**:**7006**

打印：
```
>>> Creating cluster
Connecting to node 192.168.1.1:7001: OK
Connecting to node 192.168.1.1:7002: OK
Connecting to node 192.168.1.1:7003: OK
Connecting to node 192.168.1.1:7004: OK
Connecting to node 192.168.1.1:7005: OK
Connecting to node 192.168.1.1:7006: OK
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.1.1:7001
192.168.1.1:7002
192.168.1.1:7003
Adding replica 192.168.1.1:7004 to 192.168.1.1:7001
Adding replica 192.168.1.1:7005 to 192.168.1.1:7002
Adding replica 192.168.1.1:7006 to 192.168.1.1:7003
M: 5a8523db7e12ca600dc82901ced06741b3010076 192.168.1.1:7001
   slots:0-5460 (5461 slots) master
M: bf6f0929044db485dea9b565bb51e0c917d20a53 192.168.1.1:7002
   slots:5461-10922 (5462 slots) master
M: c5e334dc4a53f655cb98fa3c3bdef8a808a693ca 192.168.1.1:7003
   slots:10923-16383 (5461 slots) master
S: 2a61b87b49e5b1c84092918fa2467dd70fec115f 192.168.1.1:7004
   replicates 5a8523db7e12ca600dc82901ced06741b3010076
S: 14848b8c813766387cfd77229bd2d1ffd6ac8d65 192.168.1.1:7005
   replicates bf6f0929044db485dea9b565bb51e0c917d20a53
S: 3192cbe437fe67bbde9062f59d5a77dabcd0d632 192.168.1.1:7006
   replicates c5e334dc4a53f655cb98fa3c3bdef8a808a693ca
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.....
>>> Performing Cluster Check (using node 192.168.1.1:7001)
M: 5a8523db7e12ca600dc82901ced06741b3010076 192.168.1.1:7001
   slots:0-5460 (5461 slots) master
M: bf6f0929044db485dea9b565bb51e0c917d20a53 192.168.1.1:7002
   slots:5461-10922 (5462 slots) master
M: c5e334dc4a53f655cb98fa3c3bdef8a808a693ca 192.168.1.1:7003
   slots:10923-16383 (5461 slots) master
M: 2a61b87b49e5b1c84092918fa2467dd70fec115f 192.168.1.1:7004
   slots: (0 slots) master
   replicates 5a8523db7e12ca600dc82901ced06741b3010076
M: 14848b8c813766387cfd77229bd2d1ffd6ac8d65 192.168.1.1:7005
   slots: (0 slots) master
   replicates bf6f0929044db485dea9b565bb51e0c917d20a53
M: 3192cbe437fe67bbde9062f59d5a77dabcd0d632 192.168.1.1:7006
   slots: (0 slots) master
   replicates c5e334dc4a53f655cb98fa3c3bdef8a808a693ca
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

### 查看Redis运行状态
```
ps -el | grep redis
5 S     0  1999     1  0  80   0 - 34359 ep_pol ?        00:00:00 redis-server
5 S     0  2003     1  0  80   0 - 34359 ep_pol ?        00:00:00 redis-server
5 S     0  2007     1  0  80   0 - 34359 ep_pol ?        00:00:00 redis-server
5 S     0  2011     1  0  80   0 - 34359 ep_pol ?        00:00:00 redis-server
5 S     0  2017     1  0  80   0 - 34359 ep_pol ?        00:00:00 redis-server
5 S     0  2023     1  0  80   0 - 34359 ep_pol ?        00:00:00 redis-server
```
## Redis集群操作

### 新增一个Master节点

/usr/local/redis-cluster/redis-trib.rb add-node **新增节点192.168.1.1:端口** **已知节点192.168.1.1:端口**
> 如192.168.1.1:6385 192.168.1.1:6379

打印：
```
>>> Adding node 新增192.168.1.1:端口 to cluster 192.168.1.1:7001
Connecting to node 192.168.1.1:7001: OK
Connecting to node 192.168.1.1:7006: OK
Connecting to node 192.168.1.1:7005: OK
Connecting to node 192.168.1.1:7004: OK
Connecting to node 192.168.1.1:7002: OK
Connecting to node 192.168.1.1:7003: OK
>>> Performing Cluster Check (using node 192.168.1.1:7001)
M: 614d0def75663f2620b6402a017014b57c912dad 192.168.1.1:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: fa299e41c173fa807ba04684c2f5e5e185d5f7d0 192.168.1.1:7006
   slots: (0 slots) slave
   replicates 83df08875c7707878756364039df0a4c8658f272
S: adb99506ddccad332e09258565f2e5f4f456a150 192.168.1.1:7005
   slots: (0 slots) slave
   replicates 8aac82b63d42a1989528cd3906579863a5774e77
S: a69b98937844c6050ee5885266ccccb185a3f36a 192.168.1.1:7004
   slots: (0 slots) slave
   replicates 614d0def75663f2620b6402a017014b57c912dad
M: 8aac82b63d42a1989528cd3906579863a5774e77 192.168.1.1:7002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: 83df08875c7707878756364039df0a4c8658f272 192.168.1.1:7003
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
Connecting to node 新增192.168.1.1:端口: OK
>>> Send CLUSTER MEET to node 新增192.168.1.1:端口 to make it join the cluster.
[OK] New node added correctly.
```
> 注意：当添加节点成功以后，新增的节点不会有任何数据，因为它没有分配任何的slot（hash槽）。我们需要为新节点手工分配slot。

#### 为新节点分配slot
/usr/local/redis-cluster/redis-trib.rb add-node **192.168.1.1:端口**(已知节点)

打印：
```
>>> Performing Cluster Check (using node 192.168.1.1:7001)
M: 614d0def75663f2620b6402a017014b57c912dad 192.168.1.1:7001
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: fa299e41c173fa807ba04684c2f5e5e185d5f7d0 192.168.1.1:7006
   slots: (0 slots) slave
   replicates 83df08875c7707878756364039df0a4c8658f272
S: adb99506ddccad332e09258565f2e5f4f456a150 192.168.1.1:7005
   slots: (0 slots) slave
   replicates 8aac82b63d42a1989528cd3906579863a5774e77
M: 382634a4025778c040b7213453fd42a709f79e28 192.168.1.1:7007
   slots: (0 slots) master
   0 additional replica(s)
S: a69b98937844c6050ee5885266ccccb185a3f36a 192.168.1.1:7004
   slots: (0 slots) slave
   replicates 614d0def75663f2620b6402a017014b57c912dad
M: 8aac82b63d42a1989528cd3906579863a5774e77 192.168.1.1:7002
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: 83df08875c7707878756364039df0a4c8658f272 192.168.1.1:7003
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
// 提示一
How many slots do you want to move (from 1 to 16384)? 输入需要分配的slot数量如：200
// 提示二 是你需要把这200个slot槽移动到那个节点上去（需要指定节点id）
What is the receiving node ID? 382634a4025778c040b7213453fd42a709f79e28//节点ID
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
// 提示三 输入all为从所有主节点3个中分别抽取响应的槽数（一共为200个槽到指定的新节点中！，并且会打印执行分片的计划。）
Source node #1:all
Ready to move 200 slots.
Source nodes:
M: 614d0def75663f2620b6402a017014b57c912dad 192.168.1.1:7001
slots:0-5460 (5461 slots) master
1 additional replica(s)
M: 8aac82b63d42a1989528cd3906579863a5774e77 192.168.1.1:7002
slots:5461-10922 (5462 slots) master
1 additional replica(s)
M: 83df08875c7707878756364039df0a4c8658f272 192.168.1.1:7003
slots:10923-16383 (5461 slots) master
1 additional replica(s)
Destination node:
M: 382634a4025778c040b7213453fd42a709f79e28 192.168.1.1:7007
slots: (0 slots) master
0 additional replica(s)
Resharding plan:（分片执行计划日志）
......
//提示四 输入yes执行
Do you want to proceed with the proposed reshard plan (yes/no)? yes
```

#### 完成 查看新增节点 slot数量

/usr/local/redis/bin/redis-cli -c -h 192.168.1.1 -p 7001

192.168.1.1:7001> cluster  nodes

### 给7007添加一个Slave节点
/usr/local/redis-cluster/redis-trib.rb add-node 192.168.1.1:7008 **192.168.1.1:7007**(Master节点)

需要执行replicate命令来指定当前节点（从节点）的主节点id为哪个。

/usr/local/redis-cluster/redis1/bin/redis-cli -c -h 192.168.1.1 -p 7008
192.168.1.1:7008> cluster replicate 382634a4025778c040b7213453fd42a709f79e28(7007的id)

192.168.1.1:7008> OK（提示OK则操作成功）

### 删除一个Slave节点(7008 Slave)
删除从节点7008，输入del-node命令，指定删除节点ip和端口，以及节点id

/usr/local/redis-cluster/redis-trib.rb del-node 192.168.1.1:7008 97b0e0115326833724eb0ffe1d0574ee34618e9f(节点id)

打印：
```
>>> Removing node 97b0e0115326833724eb0ffe1d0574ee34618e9f from cluster 192.168.1.1:7008
Connecting to node 192.168.1.1:7008: OK
Connecting to node 192.168.1.1:7003: OK
Connecting to node 192.168.1.1:7006: OK
Connecting to node 192.168.1.1:7002: OK
Connecting to node 192.168.1.1:7005: OK
Connecting to node 192.168.1.1:7001: OK
Connecting to node 192.168.1.1:7004: OK
Connecting to node 192.168.1.1:7007: OK
>>> Sending CLUSTER FORGET messages to the cluster...
>>> SHUTDOWN the node.
```
> 删除成功

### 删除一个Master节点(7007 Master)
这个步骤会相对比较麻烦一些，因为主节点的里面是有分配了slot槽的，所以我们这里必须先把7007里的slot槽放入到其他的可用主节点中去，然后再进行移除节点操作才行，不然会出现数据丢失问题。

需要先把其全部的数据（Slot槽）移动到其他节点上去（目前只能把Master的数据迁移到一个节点上，暂时做不了平均分配功能）。
> 暂不知在后续版本有无增加此功能

/usr/local/redis-cluster/redis-trib.rb reshard 192.168.1.1:7007

打印：
```
>>> Performing Cluster Check (using node 192.168.1.1:7007)
M: 382634a4025778c040b7213453fd42a709f79e28 192.168.1.1:7007
   slots:0-65,5461-5527,10923-10988 (199 slots) master
   0 additional replica(s)
S: fa299e41c173fa807ba04684c2f5e5e185d5f7d0 192.168.1.1:7006
   slots: (0 slots) slave
   replicates 83df08875c7707878756364039df0a4c8658f272
S: a69b98937844c6050ee5885266ccccb185a3f36a 192.168.1.1:7004
   slots: (0 slots) slave
   replicates 614d0def75663f2620b6402a017014b57c912dad
M: 614d0def75663f2620b6402a017014b57c912dad 192.168.1.1:7001
   slots:66-5460 (5395 slots) master
   1 additional replica(s)
M: 8aac82b63d42a1989528cd3906579863a5774e77 192.168.1.1:7002
   slots:5528-10922 (5395 slots) master
   1 additional replica(s)
S: adb99506ddccad332e09258565f2e5f4f456a150 192.168.1.1:7005
   slots: (0 slots) slave
   replicates 8aac82b63d42a1989528cd3906579863a5774e77
M: 83df08875c7707878756364039df0a4c8658f272 192.168.1.1:7003
   slots:10989-16383 (5395 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 199// 这里不一定正好200个槽
What is the receiving node ID? 614d0def75663f2620b6402a017014b57c912dad
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1:382634a4025778c040b7213453fd42a709f79e28// 这里是需要把数据移动到哪？这里输入7001的主节点id
Source node #2:done// 这里直接输入done 开始生成迁移计划
Ready to move 199 slots.
  Source nodes:
    M: 382634a4025778c040b7213453fd42a709f79e28 192.168.1.1:7007
   slots:0-65,5461-5527,10923-10988 (199 slots) master
   0 additional replica(s)
  Destination node:
    M: 614d0def75663f2620b6402a017014b57c912dad 192.168.1.1:7001
   slots:66-5460 (5395 slots) master
   1 additional replica(s)
  Resharding plan:
Moving slot 0 from 382634a4025778c040b7213453fd42a709f79e28
...
Do you want to proceed with the proposed reshard plan (yes/no)? Yes// 这里输入yes开始迁移
Moving slot 0 from 192.168.1.1:7007 to 192.168.1.1:7001:
```

使用del-node命令删除7007主节点即可

/usr/local/redis-cluster/redis-trib.rb del-node 192.168.1.1:7007 382634a4025778c040b7213453fd42a709f79e28(7007ID)
输出如下：
```
>>> Removing node 382634a4025778c040b7213453fd42a709f79e28 from cluster 192.168.1.171:7007
Connecting to node 192.168.1.171:7007: OK
Connecting to node 192.168.1.171:7006: OK
Connecting to node 192.168.1.171:7004: OK
Connecting to node 192.168.1.171:7001: OK
Connecting to node 192.168.1.171:7002: OK
Connecting to node 192.168.1.171:7005: OK
Connecting to node 192.168.1.171:7003: OK
>>> Sending CLUSTER FORGET messages to the cluster...
>>> SHUTDOWN the node。
```
#### 删除成功 查看集群状态

/usr/local/redis/bin/redis-cli -c -h 192.168.1.1 -p 7001

192.168.1.1:7001> cluster  nodes

## jedis连接
### pom坐标：
```xml
   <dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>2.7.0</version>
   </dependency>
```

### 单实例连接
```java
    // 单实例连接redis
    @Test
    public void testJedisSingle() {
        Jedis jedis = new Jedis("192.168.1.1", 7001);
        jedis.set("name", "binux");
        String name = jedis.get("name");
        System.out.println(name);
        jedis.close();
    }
```

#### 使用连接池连接
   通过单实例连接redis不能对redis连接进行共享，可以使用连接池对redis连接进行共享，提高资源利用率，使用jedisPool连接redis服务，如下代码：
```java
    // 连接池连接redis
    @Test
    public void pool() {
        JedisPoolConfig config = new JedisPoolConfig();
        //最大连接数
        config.setMaxTotal(30);
        //最大连接空闲数
        config.setMaxIdle(2);

        JedisPool pool = new JedisPool(config, "192.168.1.1", 7001);
        Jedis jedis = null;

        try  {
             jedis = pool.getResource();
             jedis.set("name", "binux");
             String name = jedis.get("name");
             System.out.println(name);
        }catch(Exception ex){
             ex.printStackTrace();
        }finally{
             if(jedis != null){
                 //关闭连接
                 jedis.close();
             }
        }
    }

```
### jedis与spring整合
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.2.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-3.2.xsd ">

    <!-- 连接池配置 -->
    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <!-- 最大连接数 -->
        <property name="maxTotal" value="30" />
        <!-- 最大空闲连接数 -->
        <property name="maxIdle" value="10" />
        <!-- 每次释放连接的最大数目 -->
        <property name="numTestsPerEvictionRun" value="1024" />
        <!-- 释放连接的扫描间隔（毫秒） -->
        <property name="timeBetweenEvictionRunsMillis" value="30000" />
        <!-- 连接最小空闲时间 -->
        <property name="minEvictableIdleTimeMillis" value="1800000" />
        <!-- 连接空闲多久后释放, 当空闲时间>该值 且 空闲连接>最大空闲连接数 时直接释放 -->
        <property name="softMinEvictableIdleTimeMillis" value="10000" />
        <!-- 获取连接时的最大等待毫秒数,小于零:阻塞不确定的时间,默认-1 -->
        <property name="maxWaitMillis" value="1500" />
        <!-- 在获取连接的时候检查有效性, 默认false -->
        <property name="testOnBorrow" value="true" />
        <!-- 在空闲时检查有效性, 默认false -->
        <property name="testWhileIdle" value="true" />
        <!-- 连接耗尽时是否阻塞, false报异常,ture阻塞直到超时, 默认true -->
        <property name="blockWhenExhausted" value="false" />
    </bean>

    <!-- redis单机 通过连接池 -->
    <bean id="jedisPool" class="redis.clients.jedis.JedisPool" destroy-method="close">
        <constructor-arg name="poolConfig" ref="jedisPoolConfig"/>
        <constructor-arg name="host" value="192.168.1.1"/>
        <constructor-arg name="port" value="7001"/>
    </bean>
</beans>

```
#### 测试代码
```java
    private ApplicationContext applicationContext;

    @Before
    public void init() {
        applicationContext = new ClassPathXmlApplicationContext("classpath:spring-redis.xml");
    }

    @Test
    public void testJedisPool() {
    JedisPool pool = (JedisPool) applicationContext.getBean("jedisPool");
             try  {
             jedis = pool.getResource();

             jedis.set("name", "binux");
             String name = jedis.get("name");
             System.out.println(name);

             }catch(Exception ex){
             ex.printStackTrace();
             }finally{
                 if(jedis != null){
                     //关闭连接
                     jedis.close();
                 }
             }
    }
```

### 多实例连接
```java
<bean id="jedisClient" class="redis.clients.jedis.JedisCluster">
        <constructor-arg name="nodes">
            <set>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="${redis.server.cluster.1}"/>
                    <constructor-arg name="port" value="${redis.server.cluster.port.1}"/>
                </bean>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="${redis.server.cluster.2}"/>
                    <constructor-arg name="port" value="${redis.server.cluster.port.2}"/>
                </bean>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="${redis.server.cluster.3}"/>
                    <constructor-arg name="port" value="${redis.server.cluster.port.3}"/>
                </bean>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="${redis.server.cluster.4}"/>
                    <constructor-arg name="port" value="${redis.server.cluster.port.4}"/>
                </bean>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="${redis.server.cluster.5}"/>
                    <constructor-arg name="port" value="${redis.server.cluster.port.5}"/>
                </bean>
                <bean class="redis.clients.jedis.HostAndPort">
                    <constructor-arg name="host" value="${redis.server.cluster.6}"/>
                    <constructor-arg name="port" value="${redis.server.cluster.port.6}"/>
                </bean>
            </set>
        </constructor-arg>
    </bean>
```

#### 测试代码
```java
    private ApplicationContext applicationContext;

    @Before
    public void init() {
        applicationContext = new ClassPathXmlApplicationContext("classpath:spring-redis.xml");
    }

    @Test
    public void testJedisPool() {
    JedisCluster jedisCluster = (JedisCluster) applicationContext.getBean("jedisClient");
             try  {

             jedisCluster.set("name", "binux");
             String name = jedisCluster.get("name");
             System.out.println(name);

             }catch(Exception ex){
             ex.printStackTrace();
             }finally{
                 if(jedis != null){
                     //关闭连接
                     jedis.close();
                 }
             }
    }
```


---

## 总结
本篇只是讲了Redis集群的搭建和集群的基本操作 想深入了解的 建议自己买本书来看看 比如[Redis设计与实现](https://www.amazon.cn/Redis/dp/B00L4XHH0S/ref=tmm_pap_swatch_0?_encoding=UTF8&qid=1488619617&sr=8-2)、[Redis实战](https://www.amazon.cn/图书/dp/B016YLS2LM/ref=sr_1_1?ie=UTF8&qid=1488619617&sr=8-1&keywords=redis)等

安装如有问题 可以在下方评论 我会尽我所能的帮你解决一些问题。


---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
