---
layout:     post
title:      "Xbin-Store(分布式商城)项目所用Linux服务系列 FastDFS安装(五)"
subtitle:   "————FastDFS安装"
date:       2017-03-05
author:     "Binux"
header-img: "img/in-post/Linux-FastDFS/blog.png"
catalog: true
tags:
    - XBin-Store
    - FastDFS
    - Linux
---

> “这篇文章将介绍如何安装FastDFS,集群有点麻烦 可能以后会写篇 毕竟我个人开发是足够的”

## 系列
* [Xbin-Store(分布式商城)项目所用Linux服务系列 MySQL安装(一)](http://binux.cn/2017/03/01/Linux-MySQL-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Redis集群安装(二)](http://binux.cn/2017/03/03/Redis-Cluster-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Zookeeper集群安装(三)](http://binux.cn/2017/03/04/Zookeeper-Cluster-Install/)
* [Xbin-Store(分布式商城)项目所用Linux服务系列 Nginx安装(四)](http://binux.cn/2017/03/04/Nginx-Install/)
* **[Xbin-Store(分布式商城)项目所用Linux服务系列 FastDFS安装(五)](http://binux.cn/2017/03/05/FastDFS-Install/)**
* [Xbin-Store(分布式商城)项目所依赖的Linux服务器 Solr集群安装(六)](http://binux.cn/2017/03/06/Solr-Cluster-Install/)
* [Xbin-Store(分布式商城)项目所依赖的Linux服务器 RocketMQ集群安装(七)](http://binux.cn/2017/03/07/RocketMQ-Cluster-Install/)

## 前言
本文基于CentOS6.5安装

### FastDFS方案

主机IP		|	名称	        	|
192.168.1.1	|	trackerd		|
192.168.1.2	|	storaged		|

### 环境准备

#### 下载软件：
* [libfastcommon-master.zip](http://download.csdn.net/detail/cynicismsrs/9771164)
* [FastDFS_v5.05.tar](http://download.csdn.net/detail/cynicismsrs/9771173)
* [fastdfs-nginx-module_v1.16.tar](http://download.csdn.net/detail/cynicismsrs/9771174)
* [fastdfs_client_v1.24.jar](http://download.csdn.net/detail/cynicismsrs/9771212)

#### 安装gcc

命令：yum install make cmake gcc gcc-c++

### 解压libfastcommon
命令：unzip libfastcommon-master.zip -d /usr/local/fast/

### 关闭防火墙
> 在学习时可以不用考虑防火墙的问题

---

## 正文

### 安装libfastcommon
```
./make.sh install
mkdir -p /usr/lib64
install -m 755 libfastcommon.so /usr/lib64
mkdir -p /usr/include/fastcommon
```
打印：
```
install -m 644 common_define.h hash.h chain.h logger.h base64.h shared_func.h pthread_func.h ini_file_reader.h _os_bits.h sockopt.h sched_thread.h http_func.h md5.h local_ip_func.h avl_tree.h ioevent.h ioevent_loop.h fast_task_queue.h fast_timer.h process_ctrl.h fast_mblock.h connection_pool.h /usr/include/fastcommon
```
> 注意安装的路径：libfastcommon默认安装到了/usr/lib64/这个位置。

### 进行软链接创建
```
ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
```
### 安装FastDFS（同时进行）

#### 解压
tar -zxvf FastDFS_v5.05.tar.gz -C /usr/local/fast/

#### 安装
```
cd /usr/local/fast/FastDFS
[root@storm1 FastDFS]# ./make.sh install
```

打印：
```
mkdir -p /usr/bin
mkdir -p /etc/fdfs
cp -f fdfs_trackerd /usr/bin
if [ ! -f /etc/fdfs/tracker.conf.sample ]; then cp -f ../conf/tracker.conf /etc/fdfs/tracker.conf.sample; fi
mkdir -p /usr/bin
mkdir -p /etc/fdfs
cp -f fdfs_storaged  /usr/bin
if [ ! -f /etc/fdfs/storage.conf.sample ]; then cp -f ../conf/storage.conf /etc/fdfs/storage.conf.sample; fi
mkdir -p /usr/bin
mkdir -p /etc/fdfs
mkdir -p /usr/lib64
cp -f fdfs_monitor fdfs_test fdfs_test1 fdfs_crc32 fdfs_upload_file fdfs_download_file fdfs_delete_file fdfs_file_info fdfs_appender_test fdfs_appender_test1 fdfs_append_file fdfs_upload_appender /usr/bin
if [ 0 -eq 1 ]; then cp -f libfdfsclient.a /usr/lib64; fi
if [ 1 -eq 1 ]; then cp -f libfdfsclient.so /usr/lib64; fi
mkdir -p /usr/include/fastdfs
cp -f ../common/fdfs_define.h ../common/fdfs_global.h ../common/mime_file_parser.h ../common/fdfs_http_shared.h ../tracker/tracker_types.h ../tracker/tracker_proto.h ../tracker/fdfs_shared_func.h ../storage/trunk_mgr/trunk_shared.h tracker_client.h storage_client.h storage_client1.h client_func.h client_global.h fdfs_client.h /usr/include/fastdfs
if [ ! -f /etc/fdfs/client.conf.sample ]; then cp -f ../conf/client.conf /etc/fdfs/client.conf.sample; fi
```

#### 服务脚本在：
```
ls /etc/init.d/ | grep fdfs
fdfs_storaged
fdfs_trackerd
```

#### 配置文件在：
```
ls /etc/fdfs/
client.conf.sample  storage.conf.sample  tracker.conf.sample
```
#### 命令行工具在/usr/bin/目录下
```
ls /usr/bin/ | grep fdfs
fdfs_appender_test
fdfs_appender_test1
fdfs_append_file
fdfs_crc32
fdfs_delete_file
fdfs_download_file
fdfs_file_info
fdfs_monitor
fdfs_storaged
fdfs_test
fdfs_test1
fdfs_trackerd
fdfs_upload_appender
fdfs_upload_file
```
> 因为FastDFS服务脚本设置的bin目录为/usr/local/bin/下,但是实际我们安装在了/u sr/bin/下面。所以我们需要修改FastDFS配置文件中的路径，也就是需要修改俩 个配置文件：
> * 命令：vim /etc/init.d/fdfs_storaged
> * 进行全局替换命令：%s+/usr/local/bin+/usr/bin
> * 命令：vim /etc/init.d/fdfs_trackerd
> * 进行全局替换命令：%s+/usr/local/bin+/usr/bin

## 配置跟踪器（trackerd）
### 修改配置文件
```
cd /etc/fdfs/
cp tracker.conf.sample tracker.conf
vim /etc/fdfs/tracker.conf
```

#### 修改
```
# the base path to store data and log files
base_path=/fastdfs/tracker
```
> * 目录命令：cd /fastdfs/tracker/ && ll
> * 启动tracker命令：/etc/init.d/fdfs_trackerd start
> * 查看进程命令：ps -el \| grep fdfs
> * 停止tracker命令：/etc/init.d/fdfs_trackerd stop

#### 查看是否启动成功
##### 启动前：
```
cd /fastdfs/tracker/ && ll
total 0
```

##### 启动
/etc/init.d/fdfs_trackerd start

##### 启动成功
```
cd /fastdfs/tracker/ && ll
total 8
drwxr-xr-x. 2 root root 4096 Dec 14 20:48 data
drwxr-xr-x. 2 root root 4096 Dec 14 20:48 logs
```
> 可以设置开机启动跟踪器：（一般生产环境需要开机启动一些服务，如keepalived、Nginx、tomcat等等）
> * 命令：vim /etc/rc.d/rc.local
> * 加入配置：/etc/init.d/fdfs_trackerd start


## 配置FastDFS存储（storaged）

### 修改配置文件storage.conf

> * 进入文件目录：cd /etc/fdfs/，进行copy storage文件一份
> * 命令：cd /etc/fdfs/
> * 命令：cp storage.conf.sample storage.conf
> * 命令：vim /etc/fdfs/storage.conf

### 修改内容：
```
base_path=/fastdfs/storage
store_path0=/fastdfs/storage
tracker_server=192.168.1.1:22122
http.server_port=8888
```
### 创建存储目录：
mkdir -p /fastdfs/storage

### 启动存储（storage）
命令：/etc/init.d/fdfs_storaged start

### 查看是否启动成功
ps -ef \| grep fdfs

> 可以设置开机启动跟踪器：（一般生产环境需要开机启动一些服务，如keepaliv ed、linux、tomcat等等）
> * 命令：vim /etc/rc.d/rc.local
> * 加入配置：/etc/init.d/fdfs_storaged start

## 测试
### 首先我们在跟踪器（tracker）里copy一份client.conf文件。
命令：cd /etc/fdfs/

命令：cp client.conf.sample client.conf
### 编辑client.conf文件
vim /etc/fdfs/client.conf
### 修改内容：
```
base_path=/fastdfs/tracker
tracker_server=**192.168.1.1**:22122
```

### 测试上传
```
/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /root/FastDFS/libfastcommon-master.zip
```

打印：

group1/M00/00/00/2-YyVlhRQlaAZLlgAAGP6hUWM6I411.zip

### 存储器查看文件
```
cd /fastdfs/tracker/data/00/00/ && ll
total 1
-rw-r--r--. 1 root root 102378 Dec 14 21:00 2-YyVlhRQlaAZLlgAAGP6hUWM6I411.zip
```

## FastDFS与Nginx整合（Storage）

### 解压Nginx
tar -zxvf nginx-1.6.2.tar.gz

### 安装fastdfs-nginx-module_v1.16.tar.gz
```
tar -zxvf FastDFS/fastdfs-nginx-module_v1.16.tar.gz -C /usr/local/fast/
cd /usr/local/fast/fastdfs-nginx-module/src/
```

### 编辑配置文件config
vim /usr/local/fast/fastdfs-nginx-module/src/config
```
前：
CORE_INCS="$CORE_INCS /usr/local/include/fastdfs /usr/local/include/fastcommon/"
后：
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
```
### FastDFS与Nginx进行集成
cd nginx-1.6.2/

./configure --add-module=/usr/local/fast/fastdfs-nginx-module/src/

make && make install

### 复制、修改配置文件
cp /usr/local/fast/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/

vim /etc/fdfs/mod_fastdfs.conf

```
connect_timeout=10
tracker_server=192.168.1.1:22122
url_have_group_name = true
store_path0=/fastdfs/storage
```

### 复制FastDFS里的2个文件
cd /usr/local/fast/FastDFS/conf/

cp http.conf mime.types /etc/fdfs/

### 建立软连接
ln -s /fastdfs/storage/data/ /fastdfs/storage/data/M00

### 修改Nginx配置
vim nginx.conf

```
listen 80;
server_name localhost;
location ~/group([0-9])/M00 {
    #alias /fastdfs/storage/data;
    ngx_fastdfs_module;
}
```

### 启动Nginx
> /usr/local/nginx/sbin/nginx

### 测试

访问 http://192.168.1.2/group1/M00/00/00/2-YyVlhRQlaAZLlgAAGP6hUWM6I411.zip


* 启动停止命令
* 启动命令： 启动tracker命令：/etc/init.d/fdfs_trackerd start
* 查看进程命令：ps -el \| grep fdfs
* 启动storage命令：/etc/init.d/fdfs_storaged start
* 查看进程命令：ps -el \| grep fdfs
* 启动nginx命令：/usr/local/nginx/sbin/nginx
* 停止命令： 停止tracker命令：/etc/init.d/fdfs_trackerd stop
* 关闭storage命令：/etc/init.d/fdfs_storaged stop
* 关闭nginx命令：/usr/local/nginx/sbin/nginx -s stop

## 使用Java客户端操作
### classpath下建立文件fastdfs_client.conf
#### 输入以下内容
tracker_server = 192.168.125.129:22122

```java
package vip.xubin.utils;

import org.apache.commons.lang3.StringUtils;
import org.apache.log4j.Logger;
import org.csource.common.NameValuePair;
import org.csource.fastdfs.*;

import java.io.*;


/**
 * FastDFS 工具类
 */
public class FastDFSClientUtils {

	private static final String CONF_FILENAME = Thread.currentThread().getContextClassLoader().getResource("fastdfs_client.conf").getPath();

	private static Logger logger = Logger.getLogger(FastDFSClientUtils.class);


	private static TrackerClient trackerClient;


	//加载文件
	static {
		try {
			ClientGlobal.init(CONF_FILENAME);
			TrackerGroup trackerGroup = ClientGlobal.g_tracker_group;
			trackerClient = new TrackerClient(trackerGroup);
		} catch (Exception e) {
			logger.error(e);
		}
	}

	/**
	 * 上传
	 * @param file 文件
	 * @param path 路径
	 * @return
	 *          上传成功返回id，失败返回null
	 */
	public static String upload(File file, String path) {
		TrackerServer trackerServer = null;
		StorageServer storageServer = null;
		StorageClient1 storageClient1 = null;
		FileInputStream fis = null;
		try {
			NameValuePair[] meta_list = null; // new NameValuePair[0];
			fis = new FileInputStream(file);
			byte[] file_buff = null;
			if (fis != null) {
				int len = fis.available();
				file_buff = new byte[len];
				fis.read(file_buff);
			}

			trackerServer = trackerClient.getConnection();
			if (trackerServer == null) {
				logger.error("getConnection return null");
			}
			storageServer = trackerClient.getStoreStorage(trackerServer);
			storageClient1 = new StorageClient1(trackerServer, storageServer);
			String fileid = storageClient1.upload_file1(file_buff, getFileExt(path), meta_list);

			return fileid;
		} catch (Exception ex) {
			logger.error(ex);
			return null;
		}finally{
			if (fis != null){
				try {
					fis.close();
				} catch (IOException e) {
					logger.error(e);
				}
			}
			if (storageServer != null){
				try {
					storageServer.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if (trackerServer != null){
				try {
					trackerServer.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			storageClient1 = null;
		}
	}

	/**
	 * 上传
	 * @param data 数据
	 * @param extName 路径
	 * @return
	 *          上传成功返回id，失败返回null
	 */
	public static String upload(byte[] data, String extName) {
		TrackerServer trackerServer = null;
		StorageServer storageServer = null;
		StorageClient1 storageClient1 = null;
		try {
			NameValuePair[] meta_list = null; // new NameValuePair[0];

			trackerServer = trackerClient.getConnection();
			if (trackerServer == null) {
				logger.error("getConnection return null");
			}
			storageServer = trackerClient.getStoreStorage(trackerServer);
			storageClient1 = new StorageClient1(trackerServer, storageServer);
			String fileid = storageClient1.upload_file1(data, extName, meta_list);
			return fileid;
		} catch (Exception ex) {
			logger.error(ex);
			return null;
		}finally{
			if (storageServer != null){
				try {
					storageServer.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if (trackerServer != null){
				try {
					trackerServer.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			storageClient1 = null;
		}
	}

	/**
	 * 下载
	 * @param fileId 文件id
	 * @return
	 *          返回InputStream
	 */
	public static InputStream download(String groupName, String fileId) {
		TrackerServer trackerServer = null;
		StorageServer storageServer = null;
		StorageClient1 storageClient1 = null;
		try {
			trackerServer = trackerClient.getConnection();
			if (trackerServer == null) {
				logger.error("getConnection return null");
			}
			storageServer = trackerClient.getStoreStorage(trackerServer, groupName);
			storageClient1 = new StorageClient1(trackerServer, storageServer);
			byte[] bytes = storageClient1.download_file1(fileId);
			InputStream inputStream = new ByteArrayInputStream(bytes);
			return inputStream;
		} catch (Exception ex) {
			logger.error(ex);
			return null;
		} finally {
			if (storageServer != null){
				try {
					storageServer.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if (trackerServer != null){
				try {
					trackerServer.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			storageClient1 = null;
		}
	}

	/**
	 * 删除
	 * @param fileId 文件id
	 * @return
	 *          删除成功返回0，非0则操作失败，返回错误代码
	 */
	public static int delete(String groupName, String fileId) {
		TrackerServer trackerServer = null;
		StorageServer storageServer = null;
		StorageClient1 storageClient1 = null;
		try {
			trackerServer = trackerClient.getConnection();
			if (trackerServer == null) {
				logger.error("getConnection return null");
			}
			storageServer = trackerClient.getStoreStorage(trackerServer, groupName);
			storageClient1 = new StorageClient1(trackerServer, storageServer);
			int result = storageClient1.delete_file1(fileId);
			return result;
		} catch (Exception ex) {
			logger.error(ex);
			return 0;
		} finally {
			if (storageServer != null){
				try {
					storageServer.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			if (trackerServer != null){
				try {
					trackerServer.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			storageClient1 = null;
		}
	}

	/**
	 * 更新文件
	 * @param oldFileId 旧文件id
	 * @param file 新文件
	 * @param path 新文件路径
	 * @return
	 *          上传成功返回id，失败返回null
	 */
	public static String modify(String oldGroupName, String oldFileId, File file, String path) {
		String fileid = null;
		try {
			// 先上传
			fileid = upload(file, path);
			if (fileid == null) {
				return null;
			}
			// 再删除
			int delResult = delete(oldGroupName, oldFileId);
			if (delResult != 0) {
				return null;
			}
		} catch (Exception ex) {
			logger.error(ex);
			return null;
		}
		return fileid;
	}

	/**
	 * 获取文件后缀名
	 * @param fileName
	 * @return
	 *          如："jpg"、"txt"、"zip" 等
	 */
	private static String getFileExt(String fileName) {
		if (StringUtils.isBlank(fileName) || !fileName.contains(".")) {
			return "";
		} else {
			return fileName.substring(fileName.lastIndexOf(".") + 1);
		}
	}
}
```
---

## 总结
FastDFS的安装算是比较麻烦的 不过按照本篇 一步一步来 是绝对可以安装成功的 集群的安装就更麻烦了 个人暂时用不到 就不安装了

---

## 著作权声明

本文首次发布于 [Binux Blog](http://binux.cn)，转载请保留以上链接
