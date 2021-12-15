# 官网Functions and Operators部分

版本：0.266

[TOC]

--------------------------------------

## 1 Comparison Functions and Operators

来自官网：[https://prestodb.io/docs/current/functions/comparison.html](https://prestodb.io/docs/current/functions/comparison.html)

### is distinct from

```sh
presto:default> select 3 is distinct from 3;
 _col0 
-------
 false 
(1 row)

Query 20211214_012451_00004_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
164ms [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select 3 is not distinct from 3;
 _col0 
-------
 true  
(1 row)

Query 20211214_013715_00012_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select null is distinct from null;
 _col0 
-------
 false 
(1 row)

Query 20211214_013739_00013_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
247ms [0 rows, 0B] [0 rows/s, 0B/s]
```

### greatest/least

```sh
presto:default> select greatest(1,2,3,4);
 _col0 
-------
     4 
(1 row)

Query 20211214_012648_00005_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
387ms [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select least(1,2,3,4);
 _col0 
-------
     1 
(1 row)

Query 20211214_013817_00014_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
175ms [0 rows, 0B] [0 rows/s, 0B/s]
```

### any/all/some

```sh
presto:default> select 'hello' = any(values 'hello','world');
 _col0 
-------
 true  
(1 row)

Query 20211214_012811_00006_dd2sy, FINISHED, 1 node
Splits: 67 total, 67 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select 21 < all(values 19,20,21);
 _col0 
-------
 false 
(1 row)

Query 20211214_012831_00007_dd2sy, FINISHED, 1 node
Splits: 18 total, 18 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select 42 >= some(select 41 union all select 42 union all select 43);
 _col0 
-------
 true  
(1 row)

Query 20211214_012906_00009_dd2sy, FINISHED, 1 node
Splits: 85 total, 85 done (100.00%)
489ms [0 rows, 0B] [0 rows/s, 0B/s]
```
### like

```sh
presto:default> select * from (values('abc'),('_cd'),('cde')) as t(name) where name like '%#_%' escape '#';
 name 
------
 _cd  
(1 row)

Query 20211214_014124_00015_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select * from (values('a%c'),('%cd'),('cde')) as t(name) where name like '%#%%' escape '#';
 name 
------
 a%c  
 %cd  
(2 rows)

Query 20211214_014226_00016_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
403ms [0 rows, 0B] [0 rows/s, 0B/s]
```

## 2 Conditional Expressions

来自官网：[https://prestodb.io/docs/current/functions/conditional.html](https://prestodb.io/docs/current/functions/conditional.html)

### if

```sh
presto:default> select if(3>2,1);
 _col0 
-------
     1 
(1 row)

Query 20211214_014557_00018_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
224ms [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select if(3<2,1,0);
 _col0 
-------
     0 
(1 row)

Query 20211214_014604_00019_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
158ms [0 rows, 0B] [0 rows/s, 0B/s]
```

### nullif

```sh
presto:default> select nullif(2,2);
 _col0 
-------
 NULL  
(1 row)

Query 20211214_015025_00021_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select nullif(2,3);
 _col0 
-------
     2 
(1 row)

Query 20211214_015031_00022_dd2sy, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
339ms [0 rows, 0B] [0 rows/s, 0B/s]
```

### try

```sh
hive (default)> create table shipping(
              > origin_state string,
              > origin_zip string,
              > packages string, 
              > total_cost string
              > );
OK
Time taken: 3.18 seconds

hive (default)> insert into shipping values ('California','94131','25','100'),('California','P332a','5','72'),('California','94025','0','155'),('New Jersey','08544','225','490');
Query ID = root_20211214095954_10726f03-d2e4-4e63-b340-59d06a382b08
Total jobs = 1
Launching Job 1 out of 1


Status: Running (Executing on YARN cluster with App id application_1639444902055_0001)

--------------------------------------------------------------------------------
        VERTICES      STATUS  TOTAL  COMPLETED  RUNNING  PENDING  FAILED  KILLED
--------------------------------------------------------------------------------
Map 1 ..........   SUCCEEDED      1          1        0        0       0       0
--------------------------------------------------------------------------------
VERTICES: 01/01  [==========================>>] 100%  ELAPSED TIME: 6.65 s     
--------------------------------------------------------------------------------
Loading data to table default.shipping
Table default.shipping stats: [numFiles=1, numRows=4, totalSize=94, rawDataSize=90]
OK
values__tmp__table__1.tmp_values_col1   values__tmp__table__1.tmp_values_col2  values__tmp__table__1.tmp_values_col3    values__tmp__table__1.tmp_values_col4
Time taken: 12.083 seconds

hive (default)> select * from shipping;
OK
shipping.origin_state   shipping.origin_zip     shipping.packages       shipping.total_cost
California      94131   25      100
California      P332a   5       72
California      94025   0       155
New Jersey      08544   225     490
Time taken: 0.117 seconds, Fetched: 4 row(s)

presto:default> select cast(origin_zip as bigint) from shipping;

Query 20211214_020812_00005_geiez, FAILED, 1 node
Splits: 17 total, 0 done (0.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

Query 20211214_020812_00005_geiez failed: Cannot cast 'P332a' to BIGINT

presto:default> select try(cast(origin_zip as bigint)) from shipping;
 _col0 
-------
 94131 
 NULL  
 94025 
  8544 
(4 rows)

Query 20211214_020833_00007_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [4 rows, 94B] [7 rows/s, 181B/s]

presto:default> select cast(total_cost as integer)/cast(packages as integer) as per_package from shipping;

Query 20211214_021355_00009_geiez, FAILED, 1 node
Splits: 17 total, 0 done (0.00%)
348ms [0 rows, 0B] [0 rows/s, 0B/s]

Query 20211214_021355_00009_geiez failed: / by zero

presto:default> select coalesce(try(cast(total_cost as integer)/cast(packages as integer)),0) as per_package from shipping;
 per_package 
-------------
           4 
          14 
           0 
           2 
(4 rows)

Query 20211214_021501_00010_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
408ms [4 rows, 94B] [9 rows/s, 230B/s]
```

## 3 Lambda Expressions

来自官网：[https://prestodb.io/docs/current/functions/lambda.html](https://prestodb.io/docs/current/functions/lambda.html)

```sh
presto:default> select filter(array[5,-6,1,7],x->x>0);
   _col0   
-----------
 [5, 1, 7] 
(1 row)

Query 20211214_022535_00015_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select filter(array[5,-6,1,7],x->sum(x));
Query 20211214_022605_00016_geiez failed: line 1:34: Lambda expression cannot contain aggregations, window functions or grouping operations: ["sum"(x)]
select filter(array[5,-6,1,7],x->sum(x))

presto:default> select filter(array[5,-6,1,7],x->x+(select 3));
Query 20211214_022626_00017_geiez failed: line 1:36: Lambda expression cannot contain subqueries
select filter(array[5,-6,1,7],x->x+(select 3))

```

## 4 Conversion Functions

来自官网：[https://prestodb.io/docs/current/functions/conversion.html](https://prestodb.io/docs/current/functions/conversion.html)

### cast/try_cast

```sh
presto:default> desc shipping;
    Column    |  Type   | Extra | Comment 
--------------+---------+-------+---------
 origin_state | varchar |       |         
 origin_zip   | varchar |       |         
 packages     | varchar |       |         
 total_cost   | varchar |       |         
(4 rows)

Query 20211214_022837_00018_geiez, FINISHED, 1 node
Splits: 19 total, 19 done (100.00%)
0:01 [4 rows, 284B] [3 rows/s, 267B/s]

presto:default> select total_cost+1 from shipping;
Query 20211214_022851_00019_geiez failed: line 1:18: '+' cannot be applied to varchar, integer
select total_cost+1 from shipping

presto:default> select cast(total_cost as integer) from shipping;
 _col0 
-------
   100 
    72 
   155 
   490 
(4 rows)

Query 20211214_023102_00020_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
329ms [4 rows, 94B] [12 rows/s, 286B/s]

presto:default> select try_cast(origin_state as integer) from shipping;
 _col0 
-------
 NULL  
 NULL  
 NULL  
 NULL  
(4 rows)

Query 20211214_023453_00030_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [4 rows, 94B] [6 rows/s, 151B/s]
```

### parse_presto_data_size

```sh
presto:default> select parse_presto_data_size('1b');
Query 20211214_023210_00021_geiez failed: Invalid data size: '1b'

presto:default> select parse_presto_data_size('1B');
 _col0 
-------
 1     
(1 row)

Query 20211214_023226_00022_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
281ms [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select parse_presto_data_size('1KB');
Query 20211214_023231_00023_geiez failed: Invalid data size: '1KB'

presto:default> select parse_presto_data_size('1kB');
 _col0 
-------
 1024  
(1 row)

Query 20211214_023234_00024_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
150ms [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select parse_presto_data_size('2.3mB');
Query 20211214_023242_00025_geiez failed: Invalid data size: '2.3mB'

presto:default> select parse_presto_data_size('2.3MB');
  _col0  
---------
 2411724 
(1 row)

Query 20211214_023250_00026_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
150ms [0 rows, 0B] [0 rows/s, 0B/s]
```

### typeof

```sh
presto:default> select typeof(12);
  _col0  
---------
 integer 
(1 row)

Query 20211214_023332_00027_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
161ms [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select typeof('cat');
   _col0    
------------
 varchar(3) 
(1 row)

Query 20211214_023338_00028_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
116ms [0 rows, 0B] [0 rows/s, 0B/s]

presto:default> select typeof(cos(2)+1.5);
 _col0  
--------
 double 
(1 row)

Query 20211214_023349_00029_geiez, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
173ms [0 rows, 0B] [0 rows/s, 0B/s]
```

## 5 Mathematical Functions and Operators

来自官网：[https://prestodb.io/docs/current/functions/math.html](https://prestodb.io/docs/current/functions/math.html)

```sh
presto> select truncate(12.3);
 _col0 
-------
 12    
(1 row)

Query 20211215_121436_00002_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select truncate(12.7);
 _col0 
-------
 12    
(1 row)

Query 20211215_121441_00003_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
179ms [0 rows, 0B] [0 rows/s, 0B/s]

# 去掉小数点右边的 1 个数字
presto> select truncate(12.123,1);
 _col0  
--------
 12.100 
(1 row)

Query 20211215_121527_00006_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
177ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select truncate(12.123,2);
 _col0  
--------
 12.120 
(1 row)

Query 20211215_121500_00004_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select truncate(12.123,3);
 _col0  
--------
 12.123 
(1 row)

Query 20211215_121519_00005_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
324ms [0 rows, 0B] [0 rows/s, 0B/s]

# 去掉小数点左边的 1 个数字
presto> select truncate(12.123,-1);
 _col0  
--------
 10.000 
(1 row)

Query 20211215_121732_00008_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select truncate(12.123,-2);
 _col0 
-------
 0.000 
(1 row)

Query 20211215_121752_00009_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
331ms [0 rows, 0B] [0 rows/s, 0B/s]
```

## 6 Bitwise Functions

```sh
# bit_count(x, bits)  bits??

```

## 7 Decimal Functions and Operators

来自官网：[https://prestodb.io/docs/current/functions/decimal.html](https://prestodb.io/docs/current/functions/decimal.html)

```sh
# decimal(n,m) 
# n -> precision
# m -> scale
presto> create table hive.default.decimal_t(d decimal(10,5));
CREATE TABLE
presto> insert into hive.default.decimal_t values (12345.12345);
INSERT: 1 row

Query 20211215_123422_00016_vfvk7, FINISHED, 3 nodes
Splits: 53 total, 53 done (100.00%)
0:07 [0 rows, 0B] [0 rows/s, 0B/s]

presto> insert into hive.default.decimal_t values (12345.1234);
INSERT: 1 row

Query 20211215_123434_00017_vfvk7, FINISHED, 3 nodes
Splits: 69 total, 69 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select * from hive.default.decimal_t;
      d      
-------------
 12345.12340 
 12345.12345 
(2 rows)

Query 20211215_123443_00018_vfvk7, FINISHED, 3 nodes
Splits: 18 total, 18 done (100.00%)
0:02 [2 rows, 571B] [0 rows/s, 253B/s]

presto> select d*2 from hive.default.decimal_t;
    _col0    
-------------
 24690.24680 
 24690.24690 
(2 rows)

Query 20211215_123649_00020_vfvk7, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
0:02 [2 rows, 571B] [1 rows/s, 346B/s]

presto> select -d from hive.default.decimal_t;
    _col0     
--------------
 -12345.12340 
 -12345.12345 
(2 rows)

Query 20211215_123709_00021_vfvk7, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
0:01 [2 rows, 571B] [2 rows/s, 783B/s]
```

## 8 String Functions and Operators

来自官网：[https://prestodb.io/docs/current/functions/string.html](https://prestodb.io/docs/current/functions/string.html)

### String Operators

```sh
presto> select * from hive.default.student;
 id |   name   
----+----------
  1 | zhangsan 
  2 | lisi     
(2 rows)

Query 20211215_123840_00027_vfvk7, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
0:01 [2 rows, 18B] [1 rows/s, 15B/s]

presto> select name || '-male'  from hive.default.student;
     _col0     
---------------
 lisi-male     
 zhangsan-male 
(2 rows)

Query 20211215_123901_00028_vfvk7, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
0:01 [2 rows, 18B] [3 rows/s, 30B/s]
```

### String Functions

```sh
presto> select chr(123);
 _col0 
-------
 {     
(1 row)

Query 20211215_124149_00031_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
128ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select concat(name,'-male') from hive.default.student;
     _col0     
---------------
 zhangsan-male 
 lisi-male     
(2 rows)

Query 20211215_124252_00032_vfvk7, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
0:01 [2 rows, 18B] [3 rows/s, 30B/s]


presto> select lpad(name,10,'abc') from hive.default.student;
   _col0    
------------
 abzhangsan 
 abcabclisi 
(2 rows)

Query 20211215_124630_00036_vfvk7, FINISHED, 2 nodes
Splits: 18 total, 18 done (100.00%)
0:01 [2 rows, 18B] [1 rows/s, 16B/s]

presto> select length(ltrim(' aa '));
 _col0 
-------
     3 
(1 row)

Query 20211215_124735_00039_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
143ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select replace('abcd','a');
 _col0 
-------
 bcd   
(1 row)

Query 20211215_124822_00040_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
133ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select replace('abcd','a','e');
 _col0 
-------
 ebcd  
(1 row)

Query 20211215_124828_00041_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
265ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select split('a|b|c','|');
   _col0   
-----------
 [a, b, c] 
(1 row)

Query 20211215_124900_00042_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
289ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select split('a|b|c','|',3);
   _col0   
-----------
 [a, b, c] 
(1 row)

Query 20211215_124922_00043_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
182ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select split('a|b|c','|',2);
  _col0   
----------
 [a, b|c] 
(1 row)

Query 20211215_124924_00044_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
141ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select split_part('a|b|c','|',1);
 _col0 
-------
 a     
(1 row)

Query 20211215_125015_00046_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
271ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select split_part('a|b|c','|',2);
 _col0 
-------
 b     
(1 row)

Query 20211215_125020_00047_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
212ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select split_to_map('a-1|b-2|c-3','|','-');
      _col0      
-----------------
 {a=1, b=2, c=3} 
(1 row)

Query 20211215_125307_00048_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
467ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select split_to_map('a-1|b-2|c-3|a-2','|','-',(k,v1,v2)->v2);
      _col0      
-----------------
 {a=2, b=2, c=3} 
(1 row)

Query 20211215_125441_00050_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
229ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select split_to_map('a-1|b-2|c-3|a-2','|','-',(k,v1,v2)->concat(v1,v2));
      _col0       
------------------
 {a=12, b=2, c=3} 
(1 row)

Query 20211215_125450_00051_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
146ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select split_to_multimap('a-1|b-2|c-3|a-2','|','-');
          _col0           
--------------------------
 {a=[1, 2], b=[2], c=[3]} 
(1 row)

Query 20211215_125606_00052_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
164ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select strpos('abedab','ab');
 _col0 
-------
     1 
(1 row)

Query 20211215_125737_00053_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select strpos('abedab','ab',2);
 _col0 
-------
     5 
(1 row)

Query 20211215_125747_00054_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
180ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select position('ab' in 'abedab');
 _col0 
-------
     1 
(1 row)

Query 20211215_125823_00055_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
207ms [0 rows, 0B] [0 rows/s, 0B/s]
```

### Unicode Functions

## 9 Regular Expression Functions

来自官网：[https://prestodb.io/docs/current/functions/regexp.html](https://prestodb.io/docs/current/functions/regexp.html)

原文含有示例

## 10 Binary Functions and Operators

来自官网：[https://prestodb.io/docs/current/functions/binary.html](https://prestodb.io/docs/current/functions/binary.html)

## 11 JSON Functions and Operators

来自官网：[https://prestodb.io/docs/current/functions/json.html](https://prestodb.io/docs/current/functions/json.html)

原文含有示例

## 12 Date and Time Functions and Operators

来自官网：[https://prestodb.io/docs/current/functions/datetime.html](https://prestodb.io/docs/current/functions/datetime.html)

### Date and Time Operators

原文含有示例

### Time Zone Conversion

原文含有示例

### Date and Time Functions

```sh
presto> select localtime;
    _col0     
--------------
 21:18:54.720 
(1 row)

Query 20211215_131854_00061_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
401ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select current_time ;
           _col0            
----------------------------
 21:19:08.666 Asia/Shanghai 
(1 row)

Query 20211215_131908_00062_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
256ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select current_timestamp; 
                 _col0                 
---------------------------------------
 2021-12-15 21:19:26.236 Asia/Shanghai 
(1 row)

Query 20211215_131926_00063_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
244ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select current_date ;
   _col0    
------------
 2021-12-15 
(1 row)

Query 20211215_131939_00064_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
422ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select to_milliseconds(interval '2' day);
   _col0   
-----------
 172800000 
(1 row)

Query 20211215_132123_00066_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
218ms [0 rows, 0B] [0 rows/s, 0B/s]

```

### Truncation Function

原文含有示例

### Interval Functions

```sh
presto> select date_add('day',2,timestamp '2012-08-08 00:00');
          _col0          
-------------------------
 2012-08-10 00:00:00.000 
(1 row)

Query 20211215_132509_00068_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
157ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select date_diff('day',timestamp '2012-08-08 00:00',timestamp '2012-08-10 00:00');
 _col0 
-------
     2 
(1 row)

Query 20211215_132600_00070_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
204ms [0 rows, 0B] [0 rows/s, 0B/s]
```

### Duration Function

原文含有示例



```sh
presto> select date_format(current_timestamp,'%a');
 _col0  
--------
 星期三 
(1 row)

Query 20211215_133023_00072_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
168ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select date_format(current_timestamp,'%c');
 _col0 
-------
 12    
(1 row)

Query 20211215_133038_00073_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
165ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select date_parse('2012-08-08','%Y-%m-%d');
          _col0          
-------------------------
 2012-08-08 00:00:00.000 
(1 row)

Query 20211215_133856_00081_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
319ms [0 rows, 0B] [0 rows/s, 0B/s]
```

### Java Date Functions

### Extraction Function

```sh
presto> select extract(year from timestamp'2012-08-08');
 _col0 
-------
  2012 
(1 row)

Query 20211215_134139_00085_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select year(timestamp'2012-08-08');
 _col0 
-------
  2012 
(1 row)

Query 20211215_134211_00087_vfvk7, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
163ms [0 rows, 0B] [0 rows/s, 0B/s]
```

### Convenience Extraction Functions