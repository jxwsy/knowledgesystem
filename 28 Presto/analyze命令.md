# analyze命令

版本：0.266

--------------------------------------

        ANALYZE table_name [ WITH ( property_name = expression [, ...] ) ]

用来收集表和列的统计信息。

对于列，只针对基本数据类型的列的统计信息。

with 子句用来指定一些针对 connector 的属性。查看所有的可用属性：

```sh
presto:default> SELECT * FROM system.metadata.analyze_properties;
 catalog_name | property_name | default_value |         type          |        description        
--------------+---------------+---------------+-----------------------+---------------------------
 hive         | partitions    |               | array(array(varchar)) | Partitions to be analyzed 
(1 row)

Query 20211211_122425_00006_tp69h, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:02 [1 rows, 60B] [0 rows/s, 29B/s]

```

上述语句只能在 hive connector 中使用。

示例：

```sh
presto:default> select * from order_table_s;
 order_id | product_name | price | deal_day 
----------+--------------+-------+----------
        1 | cellphone    |  2000 | 201901   
        2 | tv           |  3000 | 201901   
        3 | sofa         |  8000 | 201901   
        4 | cabinet      |  5000 | 201901   
        5 | bicycle      |  1000 | 201901   
        6 | truck        | 20000 | 201901   
        1 | apple        |    10 | 201902   
        2 | banana       |     8 | 201902   
        3 | milk         |    70 | 201902   
        4 | liquor       |   150 | 201902   
(10 rows)

Query 20211211_103641_00004_tp69h, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
0:12 [10 rows, 128B] [0 rows/s, 10B/s]
```

```sh
presto:default> analyze order_table_s;
ANALYZE: 10 rows

Query 20211211_103740_00005_tp69h, FINISHED, 2 nodes
Splits: 83 total, 83 done (100.00%)
0:06 [10 rows, 128B] [1 rows/s, 22B/s]
```

```sh
presto:default> analyze hive.default.order_table_s;
ANALYZE: 10 rows

Query 20211211_122822_00010_tp69h, FINISHED, 2 nodes
Splits: 83 total, 83 done (100.00%)
0:04 [10 rows, 128B] [2 rows/s, 34B/s]

```

```sh
presto:default> analyze hive.default.order_table_s with (partitions=array[array['201901'],array['201902']]);
ANALYZE: 10 rows

Query 20211211_123009_00011_tp69h, FINISHED, 2 nodes
Splits: 83 total, 83 done (100.00%)
0:03 [10 rows, 128B] [3 rows/s, 43B/s]

```

多个分区：

```sh
ANALYZE hive.default.customers WITH (partitions = ARRAY[ARRAY['CA', 'San Francisco'], ARRAY['NY', 'NY']]);
```

------------------------------------------------------------

来自官网：[https://prestodb.io/docs/0.266/sql/analyze.html](https://prestodb.io/docs/0.266/sql/analyze.html)