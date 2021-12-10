# 在 Alluxio 上运行 Apache Hive

来自官网：[https://docs.alluxio.io/os/user/2.1/cn/compute/Hive.html](https://docs.alluxio.io/os/user/2.1/cn/compute/Hive.html)

------------------------------------------------------

[TOC]

## 1 部署环境

### 1.1 前提准备

完成以下软件安装：

- JDK

- Alluxio

- Hive

- MapReduce 可以运行在 Alluxio 上

### 1.2 配置 Hive

在 shell 或 `conf/hive-env.sh` 中设置 `HIVE_AUX_JARS_PATH`

```sh
[root@bigdata101 hive-1.2.1]# cat conf/hive-env.sh
export HIVE_AUX_JARS_PATH=/opt/alluxio-2.1.0/client/alluxio-2.1.0-client.jar:${HIVE_AUX_JARS_PATH}
```

## 2 使用 Alluxio 中的文件建表

下载数据源：[https://grouplens.org/datasets/movielens/](https://grouplens.org/datasets/movielens/)

上传到 Alluxio

```sh
[root@bigdata101 alluxio-2.1.0]# bin/alluxio fs mkdir /ml-100k
Successfully created directory /ml-100k

[root@bigdata101 alluxio-2.1.0]# bin/alluxio fs copyFromLocal /root/u.user alluxio://bigdata101:19998//ml-100k
Copied file:///root/u.user to alluxio://bigdata101:19998/ml-100k
```

建内部表

```sql
CREATE TABLE u_user (
    userid INT,
    age INT,
    gender CHAR(1),
    occupation STRING,
    zipcode STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
LOCATION 'alluxio://bigdata101:19998/ml-100k';
```

建外部表

```sql
CREATE EXTERNAL TABLE u_user_e (
    userid INT,
    age INT,
    gender CHAR(1),
    occupation STRING,
    zipcode STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|'
LOCATION 'alluxio://bigdata101:19998/ml-100k';
```

查询

```sh
hive> select * from u_user limit 5;
OK
1       24      M       technician      85711
2       53      F       other   94043
3       23      M       writer  32067
4       24      M       technician      43537
5       33      F       other   15213
Time taken: 0.787 seconds, Fetched: 5 row(s)

hive> select * from u_user_e limit 5;
OK
1       24      M       technician      85711
2       53      F       other   94043
3       23      M       writer  32067
4       24      M       technician      43537
5       33      F       other   15213
Time taken: 0.111 seconds, Fetched: 5 row(s)
```

## 3 在 ALluxio 中使用已存储在于 HDFS 中的表

前提：

HDFS 集群已经是 Alluxio 根目录下的底层存储系统（例如，在 `conf/alluxio-site.properties` 中设置属性 `alluxio.master.mount.table.root.ufs=hdfs://bigdata101:9000/alluxio` ）

外部表和内部表同理

在 Hive 中建表

```sql
CREATE TABLE u_user_t (
    userid INT,
    age INT,
    gender CHAR(1),
    occupation STRING,
    zipcode STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '|';
```

导入数据

```sh
hive> load data inpath '/in/u.user' into table u_user_t;
Loading data to table default.u_user_t
Table default.u_user_t stats: [numFiles=1, totalSize=22628]
OK
Time taken: 1.171 seconds
```

查看表信息

```sh
hive> desc formatted u_user_t;
OK
# col_name              data_type               comment             
                 
userid                  int                                         
age                     int                                         
gender                  char(1)                                     
occupation              string                                      
zipcode                 string                                      
                 
...                     
Location:               hdfs://bigdata101:9000/user/hive/warehouse/u_user_t                 
```

将表数据的存储位置从 HDFS 转移到 Alluxio 中

```sh
hive> alter table u_user_t set location "alluxio://bigdata101:19998/user/hive/warehouse/u_user_t";
OK
Time taken: 0.306 seconds
```

查看表信息

```sh
hive> desc formatted u_user_t;
OK
...                    
Location:               alluxio://bigdata101:19998/user/hive/warehouse/u_user_t
```
--------------------------------------------**???**

alluxio://bigdata101:19998/user/hive/warehouse/ 目录下没有数据，日志也没报错

```sh
hive> select count(*) from u_user_t;
FAILED: SemanticException java.io.FileNotFoundException: /user/hive/warehouse/u_user_t
```

--------------------------------------------**???**

## 4 将表的元数据恢复到HDFS

将表的存储位置恢复到 HDFS 中

```sh
hive> alter table u_user_t set location "hdfs://bigdata101:9000/user/hive/warehouse/u_user_t";
OK
Time taken: 0.438 seconds
```

## 5 将 Alluxio 作为默认文件系统 

**TODO**