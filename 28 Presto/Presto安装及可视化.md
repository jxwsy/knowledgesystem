# presto安装及可视化

[TOC]

官网地址

[https://prestodb.github.io/](https://prestodb.github.io/)

下载地址

[https://repo1.maven.org/maven2/com/facebook/presto/presto-server/0.196/presto-server-0.196.tar.gz ](https://repo1.maven.org/maven2/com/facebook/presto/presto-server/0.196/presto-server-0.196.tar.gz )

## 1. Presto Server安装

下载后，将 `presto-server-0.196.tar.gz` 解压

```sh
[root@bigdata101 opt]# tar -zxvf presto-server-0.196.tar.gz
```
进入到 `/opt/presto-0.196` 目录，创建存储数据文件夹，和存储配置文件文件夹

```sh
[root@bigdata101 presto-0.196]# mkdir data

[root@bigdata101 presto-0.196]# mkdir etc
```

在 `/opt/presto-0.196/etc` 目录下添加 `jvm.config` 配置文件

```sh
[root@bigdata101 etc]# vi jvm.config
-server
-Xmx16G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
```

Presto 可以支持多个数据源，在 Presto 中配置支持 Hive 的数据源

```sh
[root@bigdata101 etc]# mkdir catalog

[root@bigdata101 etc]# vi hive.properties
connector.name=hive-hadoop2
hive.metastore.uri=thrift://bigdata101:9083
```

将 bigdata101 上的 presto 分发到 bigdata102 和 bigdata103

分发后，在三台机器上分别配置 node 属性，node id 每个节点都不一样

```sh
[root@bigdata101 etc]# vi node.properties
node.environment=production
node.id=ffffffff-ffff-ffff-ffff-ffffffffffff
node.data-dir=/opt/presto-0.196/data

[root@bigdata102 etc]# vi node.properties
node.environment=production
node.id=ffffffff-ffff-ffff-ffff-fffffffffffe
node.data-dir=/opt/presto-0.196/data

[root@bigdata103 etc]# vi node.properties
node.environment=production
node.id=ffffffff-ffff-ffff-ffff-fffffffffffd
node.data-dir=/opt/presto-0.196/data
```

在 bigdata101 上配置 coordinator，在 bigdata102 和 bigdata103 上配置 worker

```sh
[root@bigdata101 etc]# vi config.properties
coordinator=true
node-scheduler.include-coordinator=false
http-server.http.port=8881
query.max-memory=1GB
discovery-server.enabled=true
discovery.uri=http://bigdata101:8881

[root@bigdata102 etc]# vi config.properties
coordinator=false
http-server.http.port=8881
query.max-memory=2GB
discovery.uri=http://bigdata101:8881

[root@bigdata103 etc]# vi config.properties
coordinator=false
http-server.http.port=8881
query.max-memory=2GB
discovery.uri=http://bigdata101:8881
```

启动 Hive Metastore

```sh
[root@bigdata101 hive]# 
nohup bin/hive --service metastore >/dev/null 2>&1 &
```

分别在 bigdata101、bigdata102、bigdata103 上启动 Presto Server

```sh
# 前台启动Presto，控制台显示日志
[root@bigdata101 presto-0.196]# bin/launcher run
[root@bigdata102 presto-0.196]# bin/launcher run
[root@bigdata103 presto-0.196]# bin/launcher run

# 后台启动Presto
[root@bigdata101 presto-0.196]# bin/launcher start
[root@bigdata102 presto-0.196]# bin/launcher start
[root@bigdata103 presto-0.196]# bin/launcher start
```

日志查看路径 `/opt/presto-0.196/data/var/log`

## 2. Presto命令行Client安装

下载 Presto 的客户端

[https://repo1.maven.org/maven2/com/facebook/presto/presto-cli/0.196/presto-cli-0.196-executable.jar](https://repo1.maven.org/maven2/com/facebook/presto/presto-cli/0.196/presto-cli-0.196-executable.jar)

将 `presto-cli-0.196-executable.jar` 上传到 bigdata101 的 /opt/presto-0.196 文件夹下
```sh
# 前台启动Presto，控制台显示日志
[root@bigdata101 presto-0.196]# mv presto-cli-0.196-executable.jar  prestocli
[root@bigdata102 presto-0.196]# chmod +x prestocli
[root@bigdata103 presto-0.196]# ./prestocli --server bigdata101:8881 --catalog hive --schema default

```

Presto 的命令行操作，相当于 Hive 命令行操作，每个表必须要加上 schema。

	select * from schema.table limit 100

## 3. Presto可视化Client安装

将 yanagishima-18.0.zip 上传到 bigdata101 的 `/opt` 目录

解压缩

```sh
[root@bigdata101 opt]# unzip yanagishima-18.0.zip
```

进入到 `/opt/yanagishima-18.0/conf` 文件夹，编写 yanagishima.properties 配置

```sh
[root@bigdata101 conf]# vi yanagishima.properties
jetty.port=7080
presto.datasources=atiguigu-presto
presto.coordinator.server.atiguigu-presto=http://bigdata101:8881
catalog.atiguigu-presto=hive
schema.atiguigu-presto=default
sql.query.engines=presto
```

在 `/opt/yanagishima-18.0` 路径下启动 yanagishima

```sh
[root@bigdata101 yanagishima-18.0]# nohup bin/yanagishima-start.sh >y.log 2>&1 &
```

启动 web 页面

http://bigdata101:7080 


**问题：页面显不出来？**


**来源：尚硅谷数仓视频**