# presto读写alluxio中的表

版本：0.196

来自官网：[https://prestodb.io/docs/current/connector/hive.html#alluxio-configuration](https://prestodb.io/docs/current/connector/hive.html#alluxio-configuration)

------------------------------------------------------------

## 方式1：把alluxio配置文件添加到presto

The tables must be created in the Hive metastore with the `alluxio://` location prefix

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

append the Alluxio configuration directory (`${ALLUXIO_HOME}/conf`) to the Presto JVM classpath

```sh
[root@bigdata101 etc]# cat jvm.config 
...
-Xbootclasspath/a:/opt/alluxio-2.1.0/conf
```

把 alluxio-2.1.0-client.jar 复制到 presto 的 `plugin/hive-hadoop2/` 目录下

```sh
[root@bigdata101 hive-hadoop2]# pwd
/opt/presto-0.196/plugin/hive-hadoop2

[root@bigdata101 hive-hadoop2]# cp /opt/alluxio-2.1.0/client/alluxio-2.1.0-client.jar .
```

如果出现 `No FileSystem for scheme: alluxio`，说明 jar 包没复制。

把上面修改的两处同步到其他两个节点

重启，测试

```sh
[root@bigdata101 presto-0.196]# prestocli --server bigdata101:8881 --catalog hive --schema default

presto:default> select * from u_user limit 5;
 userid | age | gender | occupation | zipcode 
--------+-----+--------+------------+---------
      1 |  24 | M      | technician | 85711   
      2 |  53 | F      | other      | 94043   
      3 |  23 | M      | writer     | 32067   
      4 |  24 | M      | technician | 43537   
      5 |  33 | F      | other      | 15213   
(5 rows)

Query 20211210_151310_00006_j78ku, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
0:13 [943 rows, 22.1KB] [71 rows/s, 1.68KB/s]

```

如果出现 `failed: No worker nodes available`，在 `config.properties` 增加 `node-scheduler.include-coordinator=true`

## 方式2：使用alluxio catalog service和presto交互

In for the Alluxio Catalog to manage the metadata of other existing metastores, the other metastores must be “attached” to the Alluxio catalog. 

```sh
bin/alluxio table attachdb hive thrift://bigdata101:9083 default
```
-------------------??

报错：`Failed to connect underDb for Alluxio db 'default': Failed to get table: student error: Invalid method name: 'get_table_req'`

-------------------??

To configure the Hive connector for Alluxio Catalog Service, simply configure the connector to use the Alluxio metastore type, and provide the location to the Alluxio cluster.

```sh
connector.name=hive-hadoop2
hive.metastore=alluxio
hive.metastore.alluxio.master.address=bigdata101:19998
```
