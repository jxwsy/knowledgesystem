# hive connector procedures

版本：0.266

--------------------------------------

```sh
presto:default> show create table order_table_s;
               Create Table                
-------------------------------------------
 CREATE TABLE hive.default.order_table_s ( 
    "order_id" integer,                    
    "product_name" varchar,                
    "price" integer,                       
    "deal_day" varchar                     
 )                                         
 WITH (                                    
    format = 'TEXTFILE',                   
    partitioned_by = ARRAY['deal_day']     
 )                                         
(1 row)

Query 20211211_134942_00041_tp69h, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
264ms [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select * from order_table_s;
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

Query 20211211_135548_00043_tp69h, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
0:01 [10 rows, 128B] [11 rows/s, 143B/s]

# 添加一个空的分区
presto:default> CALL system.create_empty_partition(
             ->     schema_name => 'default',
             ->     table_name => 'order_table_s',
             ->     partition_columns => ARRAY['deal_day'],
             ->     partition_values => ARRAY['201903']);
CALL

# 查看分区
presto:default> select * from hive.default."order_table_s$partitions";
 deal_day 
----------
 201901   
 201902   
 201903   
(3 rows)

Query 20211211_142518_00076_tp69h, FINISHED, 2 nodes
Splits: 17 total, 17 done (100.00%)
0:01 [3 rows, 18B] [3 rows/s, 18B/s]
```

```sh
presto:default> CALL system.sync_partition_metadata(
             ->     schema_name => 'default', 
             ->     table_name => 'order_table_s',
             ->     mode => 'FULL',
             ->     case_sensitive => false); # false 不能加引号
CALL
```

--------------------------------------

来自官网：[https://prestodb.io/docs/current/connector/hive.html#procedures](https://prestodb.io/docs/current/connector/hive.html#procedures)