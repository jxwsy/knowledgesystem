# local file connector

版本：0.266

来自官网：[https://prestodb.io/docs/current/connector/localfile.html](https://prestodb.io/docs/current/connector/localfile.html)

--------------------------------------

作用：查询存储在每个 worker 的本地文件系统中的数据。

使用：

1. 添加 localfile.properties 文件，并分发到其他节点。重启 presto。

```sh
[root@bigdata101 ~]# cd /opt/presto-0.266/etc/catalog/
[root@bigdata101 catalog]# ls
localfile.properties
[root@bigdata101 catalog]# cat localfile.properties
connector.name=localfile
```

2. 指定 catalog 打开客户端

```sh
[root@bigdata101 presto-0.266]# prestocli --server bigdata101:8881 --catalog localfile         
presto> SHOW TABLES FROM logs;
      Table       
------------------
 http_request_log 
(1 row)

Query 20211212_081522_00003_zw5yr, FINISHED, 3 nodes
Splits: 53 total, 53 done (100.00%)
0:01 [1 rows, 30B] [0 rows/s, 24B/s]

presto> select * from localfile.logs.http_request_log limit 2;
  server_address  |        timestamp        | client_address | method |          request_uri          | user 
------------------+-------------------------+----------------+--------+-------------------------------+------
 192.168.1.6:8881 | 2021-12-12 15:58:22.449 | 192.168.1.8    | GET    | /v1/service/collector/general | NULL 
 192.168.1.6:8881 | 2021-12-12 15:58:22.448 | 192.168.1.6    | GET    | /v1/service/presto/general    | NULL 
(2 rows)

Query 20211212_081123_00002_zw5yr, FINISHED, 3 nodes
Splits: 20 total, 20 done (100.00%)
0:08 [5.68K rows, 0B] [713 rows/s, 0B/s]
```