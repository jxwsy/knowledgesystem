# mysql connector

版本：0.266

--------------------------------------

作用：可以查询和创建外部 mysql 数据库中的表。用来 join 两个不同系统(mysql\hive)间的数据，或两个不同 mysql 实例中的数据。

如果要使用多个 mysql connector，只需取不同名字。

在 mysql connector 中，不支持下面的语句：

- DELETE
- ALTER TABLE
- CREATE TABLE (CREATE TABLE AS is supported)
- GRANT
- REVOKE
- SHOW GRANTS
- SHOW ROLES
- SHOW ROLE GRANTS

使用：

1. 添加 mysql.properties 文件，并分发到其他节点。重启 presto。

```sh
[root@bigdata101 ~]# cd /opt/presto-0.266/etc/catalog/
[root@bigdata101 catalog]# ls
mysql.properties
[root@bigdata101 catalog]# cat mysql.properties
connector.name=mysql
connection-url=jdbc:mysql://bigdata101:3306
connection-user=root
connection-password=000000
```

2. 指定 catalog 打开客户端

```sh
[root@bigdata101 presto-0.266]# prestocli --server bigdata101:8881 --catalog mysql

# 查看 mysql catalog 下的所有数据库
presto> SHOW SCHEMAS FROM mysql;
       Schema       
--------------------
 azkaban            
 gmall              
 gmall_report       
 information_schema 
 metastore          
 performance_schema 
 test               
(7 rows)

Query 20211212_090238_00000_5s44w, FINISHED, 3 nodes
Splits: 53 total, 53 done (100.00%)
0:04 [7 rows, 108B] [1 rows/s, 25B/s]

# 查看 test 数据库下的所有表
presto> SHOW TABLES FROM mysql.test;
       Table        
--------------------      
 station_info          
 station_level
 ...        
(21 rows)

Query 20211212_090430_00001_5s44w, FINISHED, 3 nodes
Splits: 53 total, 53 done (100.00%)
0:02 [21 rows, 526B] [8 rows/s, 217B/s]

# 查看 station_level 表的列
presto> DESCRIBE mysql.test.station_level;
    Column    |     Type      | Extra | Comment 
--------------+---------------+-------+---------
 id           | integer       |       |         
 station_name | varchar(100) |       |         
 level        | integer  |       |         
(3 rows)

Query 20211212_090616_00002_5s44w, FINISHED, 3 nodes
Splits: 53 total, 53 done (100.00%)
0:02 [3 rows, 219B] [1 rows/s, 141B/s]

presto> SHOW COLUMNS FROM mysql.test.station_level;
    Column    |     Type      | Extra | Comment 
--------------+---------------+-------+---------
 id           | integer       |       |         
 station_name | varchar(100) |       |         
 level        | integer  |       |         
(3 rows)

Query 20211212_090710_00003_5s44w, FINISHED, 3 nodes
Splits: 53 total, 53 done (100.00%)
0:02 [3 rows, 219B] [1 rows/s, 144B/s]

presto> SELECT * FROM mysql.test.station_level limit 2;
 id |  station_name  | level 
----+----------------+-------
  1 | 崇州客运中心站 | 2     
  2 | 怀远车站       | 3    
(2 rows)

Query 20211212_090747_00005_5s44w, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
442ms [449 rows, 0B] [1.01K rows/s, 0B/s]
```

join 两个不同系统(mysql\hive)间的数据

```sh
mysql> create table product_info(
    ->     product_name varchar(10),
    ->     product_factory varchar(10)
    -> );
Query OK, 0 rows affected (0.07 sec)

mysql> insert into product_info values ("apple","aa"),("banana","bb");
Query OK, 2 rows affected (0.04 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from product_info;
+--------------+-----------------+
| product_name | product_factory |
+--------------+-----------------+
| apple        | aa              |
| banana       | bb              |
+--------------+-----------------+
2 rows in set (0.00 sec)

presto> select * from hive.default.order_table_s;
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

Query 20211212_091747_00007_5s44w, FINISHED, 3 nodes
Splits: 18 total, 18 done (100.00%)
0:02 [10 rows, 128B] [4 rows/s, 56B/s]

presto> select * from hive.default.order_table_s a 
            join mysql.test.product_info b 
            on a.product_name=b.product_name;
 order_id | product_name | price | deal_day | product_name | product_factory 
----------+--------------+-------+----------+--------------+-----------------
        1 | apple        |    10 | 201902   | apple        | aa              
        2 | banana       |     8 | 201902   | banana       | bb              
(2 rows)

Query 20211212_091643_00006_5s44w, FINISHED, 3 nodes
Splits: 163 total, 163 done (100.00%)
0:21 [12 rows, 128B] [0 rows/s, 6B/s]
```

--------------------------------------

来自官网：[https://prestodb.io/docs/0.266/connector/mysql.html](https://prestodb.io/docs/0.266/connector/mysql.html)