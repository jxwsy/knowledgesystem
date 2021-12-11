# hive connector中的支持的数据类型

版本：0.266

来自官网：[https://prestodb.io/docs/current/connector/hive.html#supported-file-types](https://prestodb.io/docs/current/connector/hive.html#supported-file-types)

来自官网：[https://prestodb.io/docs/current/connector/hive.html#avro-schema-evolution](https://prestodb.io/docs/current/connector/hive.html#avro-schema-evolution)

------------------------------------------------------------

- ORC

- Parquet

- Avro

- RCFile

- SequenceFile

- JSON

- Text

--------------------------------------------------

**Parquet**

```sh
hive> create table par_t(
    >     name string,
    >     favorite_color string,
    >     favorite_numbers array<int>
    > )
    > stored as parquet;
OK
Time taken: 0.2 seconds

hive> load data local inpath '/root/users.parquet' into table par_t;
Loading data to table default.par_t
Table default.par_t stats: [numFiles=1, totalSize=615]
OK
Time taken: 0.429 seconds

hive> select * from par_t;
OK
Alyssa  NULL    [3,9,15,20]
Ben     red     []
Time taken: 0.169 seconds, Fetched: 2 row(s)


presto:default> select * from par_t;
  name  | favorite_color | favorite_numbers 
--------+----------------+------------------
 Ben    | red            | []               
 Alyssa | NULL           | [3, 9, 15, 20]   
(2 rows)

Query 20211211_131416_00018_tp69h, FINISHED, 2 nodes
Splits: 17 total, 17 done (100.00%)
0:03 [2 rows, 728B] [0 rows/s, 274B/s]
```

**Avro**

```sh
hive> create table avro_t(
    >     name string,
    >     favorite_color string,
    >     favorite_numbers array<int>
    > )
    > stored as avro;
OK
Time taken: 0.926 seconds

hive> load data local inpath '/root/users.avro' into table avro_t;
Loading data to table default.avro_t
Table default.avro_t stats: [numFiles=1, totalSize=334]
OK
Time taken: 0.638 seconds

hive> select * from avro_t;
OK
Alyssa  NULL    [3,9,15,20]
Ben     red     []
Time taken: 0.146 seconds, Fetched: 2 row(s)

presto:default> select * from avro_t;
  name  | favorite_color | favorite_numbers 
--------+----------------+------------------
 Alyssa | NULL           | [3, 9, 15, 20]   
 Ben    | red            | []               
(2 rows)

Query 20211211_132008_00019_tp69h, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:03 [2 rows, 334B] [0 rows/s, 100B/s]
```

**Avro Schema Evolution**

It is also possible to create tables in Presto which infers the schema from a valid Avro schema file located locally or remotely in HDFS/Web server.

1. 使用 `avro_schema_url ` 指向 Avro schema

2. schema可用放在 hdfs、s3、web server 或 本地。如果放在本地，那么 hive metastore 和 presto coordinator/worker 节点要能访问到。

```sh
presto:default> CREATE TABLE hive.default.avro_data (
             ->     name varchar,
             ->     favorite_color varchar
             ->  )
             -> WITH (
             ->    format = 'AVRO',
             ->    avro_schema_url = 'hdfs://bigdata101:9000/in/user.avsc'
             -> );
CREATE TABLE

presto:default> show tables;
     Table      
----------------
 avro_data      
...       
(6 rows)

Query 20211211_133223_00023_tp69h, FINISHED, 2 nodes
Splits: 36 total, 36 done (100.00%)
0:03 [6 rows, 156B] [2 rows/s, 53B/s]

presto:default> insert into avro_data values ('zhangsan','red');
INSERT: 1 row

Query 20211211_134530_00033_tp69h, FINISHED, 2 nodes
Splits: 36 total, 36 done (100.00%)
0:02 [0 rows, 0B] [0 rows/s, 0B/s]


presto:default> select * from avro_data;
   name   | favorite_color 
----------+----------------
 zhangsan | red            
(1 row)

Query 20211211_134548_00035_tp69h, FINISHED, 2 nodes
Splits: 17 total, 17 done (100.00%)
442ms [1 rows, 241B] [2 rows/s, 544B/s]
```

3. 约定规则和限制见原文