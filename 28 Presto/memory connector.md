# memory connector

版本：0.266

--------------------------------------

作用：将数据和元数据存储在 worker 节点的 RAM 中。presto 重启后，数据就会丢失。

一些限制见原文。


使用：

1. 添加 memory.properties 文件，并分发到其他节点。重启 presto。

```sh
[root@bigdata101 ~]# cd /opt/presto-0.266/etc/catalog/
[root@bigdata101 catalog]# ls
memory.properties
[root@bigdata101 catalog]# cat memory.properties
connector.name=memory
memory.max-data-per-node=128MB  # 默认值
```

2. 指定 catalog 打开客户端

```sh
[root@bigdata101 presto-0.266]# prestocli --server bigdata101:8881 --catalog memory
presto> create table memory.default.order_table_s_mem as select * from hive.default.order_table_s;
CREATE TABLE: 10 rows

Query 20211212_085220_00002_2wn74, FINISHED, 3 nodes
Splits: 54 total, 54 done (100.00%)
0:06 [10 rows, 128B] [1 rows/s, 21B/s]

presto> select * from memory.default.order_table_s_mem;
 order_id | product_name | price | deal_day 
----------+--------------+-------+----------
        1 | apple        |    10 | 201902   
        2 | banana       |     8 | 201902   
        3 | milk         |    70 | 201902   
        4 | liquor       |   150 | 201902   
        1 | cellphone    |  2000 | 201901   
        2 | tv           |  3000 | 201901   
        3 | sofa         |  8000 | 201901   
        4 | cabinet      |  5000 | 201901   
        5 | bicycle      |  1000 | 201901   
        6 | truck        | 20000 | 201901   
(10 rows)

Query 20211212_085240_00003_2wn74, FINISHED, 3 nodes
Splits: 22 total, 22 done (100.00%)
0:01 [10 rows, 315B] [19 rows/s, 617B/s]
```

--------------------------------------

来自官网：[https://prestodb.io/docs/0.266/connector/memory.html](https://prestodb.io/docs/0.266/connector/memory.html)