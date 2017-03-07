---
layout:     post
title:      "Xbin-Store(分布式商城)项目所用Linux服务系列 Solr集群安装(六)"
subtitle:   "————Solr集群安装"
date:       2017-03-06
author:     "Binux"
header-img: "img/in-post/Linux-Solr/blog.png"
catalog: true
tags:
    - XBin-Store
    - Solr
    - Linux
---

> “这篇文章将介绍如何安装Solr集群,如何对Solr集群集群进行操作,以及使用Java客户端进行操作!”

## 系列
* [Xbin-Store(分布式商城)项目所用Linux服务系列 MySQL安装(一)](http://binux.cn/2017/03/01/Linux-MySQL-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Redis集群安装(二)](http://binux.cn/2017/03/03/Redis-Cluster-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Zookeeper集群安装(三)](http://binux.cn/2017/03/04/Zookeeper-Cluster-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Nginx安装(四)](http://binux.cn/2017/03/04/Nginx-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 FastDFS安装(五)](http://binux.cn/2017/03/05/FastDFS-Install/)
* **[Xbin-Store(分布式商城)项目所依赖的Linux服务系列 Solr集群安装(六)](http://binux.cn/2017/03/06/Solr-Cluster-Install/)**
* [Xbin-Store(分布式商城)项目所依赖的Linux服务系列 RocketMQ集群安装(七)](http://binux.cn/2017/03/07/RocketMQ-Cluster-Install/)

## 前言
### 所用虚拟机
CentOS 6.5

### IP

<img class="shadow" src="/img/in-post/Linux-Solr/img1.png" />
<small class="img-hint">IP</small>

### 下载软件
* [apache-tomcat-7.0.47.tar.gz](http://download.csdn.net/detail/cynicismsrs/9772212)
* [zookeeper-3.4.6](http://apache.fayea.com/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz)
* [solr-4.10.3.tgz](https://archive.apache.org/dist/lucene/solr/4.10.3/solr-4.10.3.tgz)
* [IK Analyzer 2012FF_hf1.zip](http://download.csdn.net/detail/zhuzhenlong/9733932)

---

## 正文

### 安装

#### 安装JDK 略

#### Zookeeper集群安装

[Xbin-Store(分布式商城)项目所用Linux服务系列 Zookeeper集群安装(三)](http://binux.cn/2017/03/04/Zookeeper-Cluster-Install/)

#### Tomcat安装(四台同时)

##### 解压

tar -zxvf apache-tomcat-7.0.47.tar.gz

##### 把解压后的Tomcat复制
/usr/local/solr/tomcat

##### 修改端口号

vim /usr/local/solr/tomcat/conf/server/server.xml

改为80端口
```xml
71行
<Connector port="80" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

##### 把solr的压缩包上传到服务器。并解压。

#### Solr安装

##### 复制war包
把/root/solr-4.10.3/dist/solr-4.10.3.war包部署到tomcat/webapps/下。并改名为ROOT.war

rm -rf /usr/local/solr/tomcat/webapps/ROOT

启动tomcat自动解压。关闭tomcat。删除ROOT.war.

##### 复制Jar包
把/root/solr-4.10.3/example/lib/ext 目录下所有的jar包复制到solr工程中。

cp /root/solr-4.10.3/example/lib/ext/* /usr/local/solr/tomcat/webapps/ROOT/WEB-INF/lib/

##### 创建solrhome

> Solrhome是存放solr服务器所有配置文件的目录。

cp -r /root/solr-4.10.3/example/solr /usr/local/solr/solrhome

##### 修改Solr配置文件

vim /usr/local/solr/tomcat/webapps/ROOT/WEB-INF/web.xml


```bash
第41行
<env-entry>
   <env-entry-name>solr/home</env-entry-name>
   <env-entry-value>/usr/local/solr/solrhome</env-entry-value>
   <env-entry-type>java.lang.String</env-entry-type>
</env-entry>
```

#### 配置中文分词

##### 解压
IK Analyzer 2012FF_hf1.zip

##### 复制

cp IKAnalyzer2012FF_u1.jar /usr/local/solr/tomcat/webapps/ROOT/WEB-INF/lib/

cp IKAnalyzer.cfg.xml ext_stopword.dic mydict.dic /usr/local/solr/tomcat/webapps/solr/WEB-INF/classes

> 扩展词典及停用词词典的字符集必须是utf-8。不能使用windows记事本编辑。

##### 配置fieldType

vim /usr/local/solr/solrhome/collection1/conf/schema.xml

> 跳转到文档末尾：G

```xml
<fieldType name="text_ik" class="solr.TextField">
 <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>
</fieldType>
```

##### 业务字段配置

> 业务字段判断标准：
> * 在搜索时是否需要在此字段上进行搜索。例如：商品名称、商品的卖点、商品的描述
> * 后续的业务是否需要用到此字段。例如：商品id。

如：
```xml
<field name="item_title" type="text_ik" indexed="true" stored="true"/>
<field name="item_price" type="long" indexed="true" stored="true"/>
<field name="item_image" type="string" indexed="false" stored="true" />
等......
```

#### 配置Solr集群

##### 上传配置文件
cd solr-4.10.3/example/scripts/cloud-scripts/

./zkcli.sh -zkhost 192.168.1.1:2181,192.168.1.2:2181,192.168.1.3:2181 -cmd upconfig -confdir /usr/local/solr/solrhome/collection1/conf -confname myconf

##### 登陆zookeeper服务器查询配置文件：
./zkCli.sh


##### 修改solr.xml监控端口为80端口

vim /usr/local/solr/solrhome/solr.xml

```xml
31行:
<solrcloud>
<str name="host">${host:}</str>
<int name="hostPort">${jetty.port:80}</int>
<str name="hostContext">${hostContext:solr}</str>
<int name="zkClientTimeout">${zkClientTimeout:30000}</int>
<bool name="genericCoreNodeNames">${genericCoreNodeNames:true}</bool>
</solrcloud>
```

##### 和zookeeper关联

修改每一台solr的tomcat 的 bin目录下catalina.sh文件中加入DzkHost指定zookeeper服务器地址：
JAVA_OPTS="-DzkHost=192.168.1.1:2181,192.168.1.2:2181,192.168.1.3:2181"

> 可以使用vim的查找功能查找到JAVA_OPTS的定义的位置

#### 启动
启动每一台solr的tomcat服务

#### 集群配置

访问：http://192.168.1.11/admin/collections?action=CREATE&name=collection2&numShards=2&replicationFactor=2

> 新建collection2将集群分为两片，每片两个副本。

访问：http://192.168.1.11/admin/collections?action=DELETE&name=collection1

> 删除 collection1

## solrJ访问solrCloud
```java
public class SolrCloudTest {
    // zookeeper地址
    private static String zkHostString = "192.168.1.1:2181,192.168.1.2:2181,192.168.1.3:2181";
    // collection默认名称
    private static String defaultCollection = "collection2";
 
    // cloudSolrServer对象
    private CloudSolrServer cloudSolrServer;
 
    // 测试方法之前构造 CloudSolrServer
    @Before
    public void init() {
        cloudSolrServer = new CloudSolrServer(zkHostString);
        cloudSolrServer.setDefaultCollection(defaultCollection);
        cloudSolrServer.connect();
    }
 
    // 向solrCloud上创建索引
    @Test
    public void testCreateIndexToSolrCloud() throws SolrServerException,
             IOException {
 
        SolrInputDocument document = new SolrInputDocument();
        document.addField("id", "100001");
        document.addField("title", "李四");
        cloudSolrServer.add(document);
        cloudSolrServer.commit();
 
    }
 
    // 搜索索引
    @Test
    public void testSearchIndexFromSolrCloud() throws Exception {
 
        SolrQuery query = new SolrQuery();
        query.setQuery("*:*");
        try {
             QueryResponse response = cloudSolrServer.query(query);
             SolrDocumentList docs = response.getResults();
 
             System.out.println("文档个数：" + docs.getNumFound());
             System.out.println("查询时间：" + response.getQTime());
 
             for (SolrDocument doc : docs) {
                 ArrayList title = (ArrayList) doc.getFieldValue("title");
                 String id = (String) doc.getFieldValue("id");
                 System.out.println("id: " + id);
                 System.out.println("title: " + title);
                 System.out.println();
             }
        } catch (SolrServerException e) {
             e.printStackTrace();
        } catch (Exception e) {
             System.out.println("Unknowned Exception!!!!");
             e.printStackTrace();
        }
    }
 
    // 删除索引
    @Test
    public void testDeleteIndexFromSolrCloud() throws SolrServerException, IOException {
 
        // 根据id删除
        UpdateResponse response = cloudSolrServer.deleteById("zhangsan");
        // 根据多个id删除
        // cloudSolrServer.deleteById(ids);
        // 自动查询条件删除
        // cloudSolrServer.deleteByQuery("product_keywords:教程");
        // 提交
        cloudSolrServer.commit();
    }
}
```


---

## 总结
solr 主要是用来搜索的 不需要自己在另外写 集群的安装也不算太麻烦 就是机器用的多了点 要7台虚拟机 实在没有这么多虚拟机的可以在一台机器搭伪分布式

---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
