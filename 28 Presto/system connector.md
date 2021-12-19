# system connector

版本：0.266

--------------------------------------

作用：提供当前运行的 presto 集群的信息和度量。

system connector 中的表介绍见原文。

无需配置，直接可以使用。

```sh
presto> SHOW SCHEMAS FROM system;
       Schema       
--------------------
 information_schema 
 jdbc               
 metadata           
 runtime            
(4 rows)

Query 20211212_092018_00008_5s44w, FINISHED, 3 nodes
Splits: 53 total, 53 done (100.00%)
0:01 [4 rows, 57B] [4 rows/s, 66B/s]

presto> SHOW TABLES FROM system.runtime;
    Table     
--------------
 nodes        
 queries      
 tasks        
 transactions 
(4 rows)

Query 20211212_092104_00009_5s44w, FINISHED, 3 nodes
Splits: 53 total, 53 done (100.00%)
0:01 [4 rows, 97B] [5 rows/s, 133B/s]

presto> SELECT * FROM system.runtime.nodes;
               node_id                |        http_uri         | node_version  | coordinator | state  
--------------------------------------+-------------------------+---------------+-------------+--------
 ffffffff-ffff-ffff-ffff-fffffffffffd | http://192.168.1.8:8881 | 0.266-c1e2e77 | false       | active 
 ffffffff-ffff-ffff-ffff-fffffffffffe | http://192.168.1.7:8881 | 0.266-c1e2e77 | false       | active 
 ffffffff-ffff-ffff-ffff-ffffffffffff | http://192.168.1.6:8881 | 0.266-c1e2e77 | true        | active 
(3 rows)

Query 20211212_092113_00010_5s44w, FINISHED, 2 nodes
Splits: 17 total, 17 done (100.00%)
279ms [3 rows, 237B] [10 rows/s, 849B/s]
```

根据 query_id 杀掉一个查询。

```sh
presto> CALL system.runtime.kill_query(query_id => '20211212_092113_00010_5s44w', message => 'Using too many resources');
Query 20211212_092324_00011_5s44w failed: Target query is not running: 20211212_092113_00010_5s44

```

--------------------------------------

来自官网：[https://prestodb.io/docs/current/connector/system.html](https://prestodb.io/docs/current/connector/system.html)