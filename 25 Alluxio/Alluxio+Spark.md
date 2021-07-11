# Alluxio+Spark

[TOC]

## 1、前提

- 已安装 Hadoop:

	使用 HDFS 作为底层存储。

	[https://github.com/ZGG2016/knowledgesystem/blob/master/06%20Hadoop/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%EF%BC%9A%E5%AE%8C%E5%85%A8%E5%88%86%E5%B8%83.md](https://github.com/ZGG2016/knowledgesystem/blob/master/06%20Hadoop/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%EF%BC%9A%E5%AE%8C%E5%85%A8%E5%88%86%E5%B8%83.md)

- 已安装 Spark

	[https://github.com/ZGG2016/knowledgesystem/blob/master/13%20Spark/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%EF%BC%9A%E5%8D%95%E6%9C%BA%E3%80%81%E4%BC%AA%E5%88%86%E5%B8%83%E3%80%81%E5%AE%8C%E5%85%A8%E5%88%86%E5%B8%83%E3%80%81%E9%AB%98%E5%8F%AF%E7%94%A8.md](https://github.com/ZGG2016/knowledgesystem/blob/master/13%20Spark/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%EF%BC%9A%E5%8D%95%E6%9C%BA%E3%80%81%E4%BC%AA%E5%88%86%E5%B8%83%E3%80%81%E5%AE%8C%E5%85%A8%E5%88%86%E5%B8%83%E3%80%81%E9%AB%98%E5%8F%AF%E7%94%A8.md)

## 2、配置

将 Alluxio Client 的 Jar 包包含在各个 Spark 节点的 classpaths 中。

```sh
[root@node1 ~]# cd /opt/alluxio-2.1.0/client/

[root@node1 client]# ls
alluxio-2.1.0-client.jar  presto

[root@node1 client]# cp alluxio-2.1.0-client.jar /opt/spark-3.0.2-bin-hadoop2.7/jars/

[root@node1 client]# scp alluxio-2.1.0-client.jar  node2:/opt/spark-3.0.2-bin-hadoop2.7/jars/
alluxio-2.1.0-client.jar   
..... 
```

或者，在运行 Spark 的每个节点上，将以下几行添加到 `spark/conf/spark-defaults.conf` 中。

	spark.driver.extraClassPath   /<PATH_TO_ALLUXIO>/client/alluxio-2.1.0-client.jar

	spark.executor.extraClassPath /<PATH_TO_ALLUXIO>/client/alluxio-2.1.0-client.jar

## 3、测试

### 3.1、使用 Alluxio 作为输入和输出

```sh
[root@node1 opt]# alluxio fs cat /wc.txt
aa
bb
aa
cc

# 计算
scala> val s = sc.textFile("alluxio://node1:19998/wc.txt")
s: org.apache.spark.rdd.RDD[String] = alluxio://node1:19998/wc.txt MapPartitionsRDD[1] at textFile at <console>:24

scala> val double = s.map(line => line + line)
double: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[2] at map at <console>:25

scala> double.saveAsTextFile("alluxio://node1:19998/wc-out")
21/04/04 16:12:00 WARN hadoop.AbstractFileSystem: delete failed: Path "/wc-out/.spark-staging-3" does not exist.

# 计算后查看
[root@node1 opt]# alluxio fs ls /wc-out
-rw-r--r--  root           root                         0       PERSISTED 04-04-2021 16:12:00:584 100% /wc-out/_SUCCESS
-rw-r--r--  root           root                        20       PERSISTED 04-04-2021 16:12:00:254 100% /wc-out/part-00000
[root@node1 opt]# alluxio fs cat /wc-out/part-00000
aaaa
bbbb
aaaa
cccc
```

### 3.2、访问底层存储中的数据

给出准确路径后，Alluxio 支持透明地从底层存储系统中获取数据。

在本节中，使用 HDFS 作为分布式存储系统的示例，即输入文件 wcc.txt 存储在 HDFS 中，而不在 Alluxio。

```sh
[root@node1 ~]# hadoop fs -cat /in/wcc.txt
hadoop
spark
hadoop

# 计算
scala> val ss = sc.textFile("alluxio://node1:19998/in/wcc.txt")
ss: org.apache.spark.rdd.RDD[String] = alluxio://node1:19998/in/wcc.txt MapPartitionsRDD[5] at textFile at <console>:24

scala> val doubles = s.map(line => line + line)
doubles: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[6] at map at <console>:25

scala> doubles.saveAsTextFile("alluxio://node1:19998/wcc-out")
21/04/04 16:19:36 WARN hadoop.AbstractFileSystem: delete failed: Path "/wcc-out/.spark-staging-7" does not exist.

# 计算后查看
[root@node1 ~]# hadoop fs -ls /
Found 6 items
drwxr-xr-x   - root supergroup          0 2021-04-04 16:19 /alluxio
drwxr-xr-x   - root supergroup          0 2021-04-04 16:17 /in
drwxr-xr-x   - root supergroup          0 2021-04-04 13:15 /mr-history
drwxr-xr-x   - root supergroup          0 2021-04-04 13:15 /out
drwx------   - root supergroup          0 2021-04-04 13:15 /tmp
drwxr-xr-x   - root supergroup          0 2021-04-04 15:25 /user

[root@node1 ~]# alluxio fs ls /
-rw-r--r--  root           root                        12       PERSISTED 04-04-2021 13:40:36:100 100% /wc.txt
drwxr-xr-x  root           root                         2       PERSISTED 04-04-2021 16:19:36:347  DIR /wcc-out

[root@node1 ~]# alluxio fs cat /wcc-out/part-00000
aaaa
bbbb
aaaa
cccc
```