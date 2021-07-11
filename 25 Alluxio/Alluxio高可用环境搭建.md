# Alluxio高可用环境搭建

[TOC]

## 1、节点角色

采用 3 台虚拟机

主机名  |     IP地址       |  角色
---|:---|:---
node1  | 192.168.xxx.xx1  |  master
node2  | 192.168.xxx.xx2  |  master
node3  | 192.168.xxx.xx3  |  worker


## 2、软件版本

软件	      |    版本
---|:---
JDK       |   jdk1.8.0_281
HADOOP    |	  hadoop-2.7.3
ALLUXIO   |   alluxio-2.1.0
ZOOKEEPER |   zookeeper-3.4.6

## 3、准备工作

### 3.1、安装 Hadoop

完全分布式安装：[https://github.com/ZGG2016/knowledgesystem/blob/master/06%20Hadoop/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%EF%BC%9A%E5%AE%8C%E5%85%A8%E5%88%86%E5%B8%83.md](https://github.com/ZGG2016/knowledgesystem/blob/master/06%20Hadoop/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%EF%BC%9A%E5%AE%8C%E5%85%A8%E5%88%86%E5%B8%83.md)

### 3.2、安装 Zookeeper

完全分布式安装：[https://github.com/ZGG2016/knowledgesystem/blob/master/19%20Zookeeper/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.md](https://github.com/ZGG2016/knowledgesystem/blob/master/19%20Zookeeper/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.md)

## 4、安装 Alluxio

在 [前述配置](https://github.com/ZGG2016/knowledgesystem/blob/master/25%20Alluxio/Alluxio%2BHDFS%2BMapReduce.md) 的基础上进行安装。

对于 node1 节点，配置 `alluxio-site.properties` 文件，并分发到其他节点。

```sh
[root@node1 conf]# cat alluxio-site.properties
alluxio.master.hostname=node1
alluxio.master.mount.table.root.ufs=hdfs://node1:9000/alluxio

alluxio.zookeeper.enabled=true
alluxio.zookeeper.address=node1:2181,node2:2181,node3:2181

alluxio.master.journal.type=UFS
# 指定正确的日志文件夹，这里使用 HDFS 来存放日志
alluxio.master.journal.folder=hdfs://node1:9000/alluxio/journal
alluxio.master.embedded.journal.addresses=node2:19200,node3:19200

alluxio.underfs.hdfs.configuration=/opt/hadoop-2.7.3/etc/hadoop/core-site.xml:/opt/hadoop-2.7.3/etc/hadoop/hdfs-site.xml:/opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml:/opt/hadoop-2.7.3/etc/hadoop/mapred-site.xml:/opt/hadoop-2.7.3/hadoop/yarn-site.xml

[root@node1 conf]# scp alluxio-site.properties node2:/opt/alluxio-2.1.0/conf/
....
```

对于 node1 节点，配置 `alluxio-env.sh` 文件，并分发到其他节点。

```sh
[root@node1 conf]# cat alluxio-env.sh
....

export JAVA_HOME=/opt/jdk1.8.0_281
export ALLUXIO_HOME=/opt/alluxio-2.1.0

[root@node1 conf]# scp alluxio-env.sh node2:/opt/alluxio-2.1.0/conf/
....
```

对于 node1 节点，配置 masters 和 workers 文件，并分发到其他节点。

```sh
[root@node1 conf]# cat masters
node1
node2

[root@node1 conf]# cat workers
node3
```

在 node1 节点上，使用 `bin/alluxio format` 命令进行格式化。

```sh
[root@node1 alluxio-2.1.0]# bin/alluxio format
Executing the following command on all worker nodes and logging to /opt/alluxio-2.1.0/logs/task.log: /opt/alluxio-2.1.0/bin/alluxio formatWorker
Waiting for tasks to finish...
All tasks finished
Formatting Alluxio Master @ node1
....
2021-04-05 15:06:53,144 INFO  Format - Formatting complete
```

分别启动各个节点

```sh
[root@node1 alluxio-2.1.0]# alluxio-start.sh master
Killed 0 process(es) on node1
Starting master @ node1. Logging to /opt/alluxio-2.1.0/logs
--- [ OK ] The master service @ node1 is in a healthy state.

[root@node1 alluxio-2.1.0]# alluxio-start.sh job_master
Killed 0 process(es) on node1
Starting job master @ node1. Logging to /opt/alluxio-2.1.0/logs
--- [ OK ] The job_master service @ node1 is in a healthy state.

[root@node1 alluxio-2.1.0]# alluxio-start.sh proxy
Killed 0 process(es) on node1
Starting proxy @ node1. Logging to /opt/alluxio-2.1.0/logs
--- [ OK ] The proxy service @ node1 is in a healthy state.
```

```sh
[root@node2 ~]# alluxio-start.sh master
Killed 0 process(es) on node2
Starting master @ node2. Logging to /opt/alluxio-2.1.0/logs
--- [ OK ] The master service @ node2 is in a healthy state.

[root@node2 ~]# alluxio-start.sh job_master
Killed 0 process(es) on node2
Starting job master @ node2. Logging to /opt/alluxio-2.1.0/logs
--- [ OK ] The job_master service @ node2 is in a healthy state.

[root@node2 ~]# alluxio-start.sh proxy
Killed 0 process(es) on node2
Starting proxy @ node2. Logging to /opt/alluxio-2.1.0/logs
--- [ OK ] The proxy service @ node2 is in a healthy state
```

```sh
# 如果前面执行了 `bin/alluxio-mount.sh Mount workers`,此处就不用了加 `Mount` 项
[root@node3 ~]# alluxio-start.sh worker Mount
Killed 0 process(es) on node3
Ramdisk /mnt/ramdisk already mounted. Skipping mounting procedure.
Starting worker @ node3. Logging to /opt/alluxio-2.1.0/logs
--- [ OK ] The worker service @ node3 is in a healthy state.

[root@node3 ~]# alluxio-start.sh job_worker
Killed 0 process(es) on node3
Starting job worker @ node3. Logging to /opt/alluxio-2.1.0/logs
--- [ OK ] The job_worker service @ node3 is in a healthy state.

[root@node3 ~]# alluxio-start.sh proxy
Killed 0 process(es) on node3
Starting proxy @ node3. Logging to /opt/alluxio-2.1.0/logs
--- [ OK ] The proxy service @ node3 is in a healthy state.
```

## 5、测试

运行测试

```sh
[root@node1 alluxio-2.1.0]# bin/alluxio fs mkdir /test
Successfully created directory /test

[root@node1 alluxio-2.1.0]# alluxio fs ls /
drwxr-xr-x  root           supergroup                   0       PERSISTED 04-05-2021 15:06:53:088  DIR /journal
drwxr-xr-x  root           root                         0   NOT_PERSISTED 04-05-2021 15:41:47:255  DIR /test

# 在 Alluxio 中读写示例文件
[root@node1 alluxio-2.1.0]# bin/alluxio runTests
....
Passed the test!

[root@node1 alluxio-2.1.0]# alluxio fs ls /
drwxr-xr-x  root           root                        24       PERSISTED 04-05-2021 15:42:30:423  DIR /default_tests_files
drwxr-xr-x  root           supergroup                   0       PERSISTED 04-05-2021 15:06:53:088  DIR /journal
drwxr-xr-x  root           root                         0   NOT_PERSISTED 04-05-2021 15:41:47:255  DIR /test
```

高可用测试

```sh
[root@node1 alluxio-2.1.0]# alluxio fs leader
node1

[root@node1 alluxio-2.1.0]# jps
41667 AlluxioJobMaster
28197 SparkSubmit
27334 NameNode
42742 Jps
27607 ResourceManager
k31143 QuorumPeerMain
42093 AlluxioProxy
41214 AlluxioMaster

# kill 掉 AlluxioMaster
[root@node1 alluxio-2.1.0]# kill -9 41214 

# 还在切换，能需要一段时间以等待新的leader当选
[root@node1 alluxio-2.1.0]# alluxio fs leader
node1
The leader is not currently serving requests.

[root@node1 alluxio-2.1.0]# alluxio fs leader
node2
```

## 6、问题

问题一：配置完成后，如果直接执行 ` bin/alluxio-start.sh all` 命令启动，那么会出现 `WARN  RetryUtils - Failed to load cluster default configuration with master (attempt 27): alluxio.exception.status.UnavailableException: Failed to handshake with master node1:19998 to load cluster default configuration values: UNAVAILABLE: io exception`

问题二：如果出现 `ERROR AlluxioMaster - Fatal error: Failed to create master process
java.lang.IllegalStateException: Raft-based embedded journal and Zookeeper cannot be used at the same time.` 错误，是 `alluxio-site.properties` 文件配置错误，可能是 `alluxio.master.hostname` 写成了 `node1,node2`，或漏掉了 `alluxio.master.embedded.journal.addresses=node2:19200,node3:19200` 配置。