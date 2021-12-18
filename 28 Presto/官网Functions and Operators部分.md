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

TODO

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

TODO

## 13 Aggregate Functions

来自官网：[https://prestodb.io/docs/current/functions/aggregate.html](https://prestodb.io/docs/current/functions/aggregate.html)

### General Aggregate Functions

```sh
presto> select count(*) from (values 1,2,null,3) as t(id);
 _col0 
-------
     4 
(1 row)

Query 20211218_015350_00021_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
345ms [0 rows, 0B] [0 rows/s, 0B/s]

# count(x) → bigint
# Returns the number of non-null input values.
presto> select count(id) from (values 1,2,null,3) as t(id);
 _col0 
-------
     3 
(1 row)

Query 20211218_015154_00020_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
215ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select sum(id) from (values 1,2,null,3) as t(id);
 _col0 
-------
     6 
(1 row)

Query 20211218_012656_00002_kjjye, FINISHED, 1 node
Splits: 18 total, 18 done (100.00%)
0:04 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select avg(id) from (values 1,2,null,3) as t(id);
 _col0 
-------
   2.0 
(1 row)

Query 20211218_012916_00005_kjjye, FINISHED, 1 node
Splits: 18 total, 18 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]
```

```sh
presto> select array_agg(id) from (values 1,2,3,4) as t(id);
    _col0     
--------------
 [1, 2, 3, 4] 
(1 row)

Query 20211218_013410_00006_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
0:02 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_agg(id order by id desc) from (values 1,2,3,4) as t(id);
    _col0     
--------------
 [4, 3, 2, 1] 
(1 row)

Query 20211218_023046_00046_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
387ms [0 rows, 0B] [0 rows/s, 0B/s]
```

```sh
# avg(time interval type) → time interval type#
#   Returns the average interval length of all input values.
# 支持 avg(interval day to second) , avg(interval year to month) 两种类型
# 2 和 3 的均值取 2，小数 0.5 换算到月份上  
presto> select avg(time) from (values interval '2-1' year to month,interval '3-1' year to month) as t(time);
 _col0 
-------
 2-7   
(1 row)

Query 20211218_014727_00017_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select avg(time) from (values interval '2-1' year to month,interval '4-1' year to month) as t(time);
 _col0 
-------
 3-1   
(1 row)

Query 20211218_014849_00018_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]
```

```sh
presto> select count_if(id>1) from (values 1,2,0,3) as t(id);
 _col0 
-------
     2 
(1 row)

Query 20211218_020306_00037_kjjye, FINISHED, 1 node
Splits: 18 total, 18 done (100.00%)
317ms [0 rows, 0B] [0 rows/s, 0B/s]

# 对于多个(x,y)形式的数据，返回最大的y对应的x
presto> select max_by(id,cnt) from (values (1,45),(2,11),(3,30)) as t(id,cnt);
 _col0 
-------
     1 
(1 row)

Query 20211218_020915_00039_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
293ms [0 rows, 0B] [0 rows/s, 0B/s]

# 返回前 n 个值
presto> select max_by(id,cnt,2) from (values (1,45),(2,11),(3,30)) as t(id,cnt);
 _col0  
--------
 [1, 3] 
(1 row)

Query 20211218_021020_00040_kjjye, FINISHED, 1 node
Splits: 18 total, 18 done (100.00%)
459ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select max(id,2) from (values 1,2,3) as t(id);
 _col0  
--------
 [3, 2] 
(1 row)

Query 20211218_021151_00043_kjjye, FINISHED, 1 node
Splits: 18 total, 18 done (100.00%)
417ms [0 rows, 0B] [0 rows/s, 0B/s]
```

```sh
presto> SELECT id, reduce_agg(value, 2, (a, b) -> a + b, (a, b) -> a + b)
     -> FROM (
     ->     VALUES
     ->         (1, 2),
     ->         (1, 3),
     ->         (1, 4),
     ->         (2, 20),
     ->         (2, 30),
     ->         (2, 40)
     -> ) AS t(id, value)
     -> GROUP BY id;
 id | _col1 
----+-------
  2 |    92 
  1 |    11 
(2 rows)

Query 20211218_023252_00047_kjjye, FINISHED, 1 node
Splits: 33 total, 33 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select set_agg(id) from (values 1,1,2,2,3,1) as t(id);
   _col0   
-----------
 [1, 2, 3] 
(1 row)

Query 20211218_023644_00050_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
236ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select set_union(arrs) from (values array[1,2,3],array[2,3,4]) as t(arrs);
    _col0     
--------------
 [1, 2, 3, 4] 
(1 row)

Query 20211218_023759_00052_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

```

### Bitwise Aggregate Functions

TODO

### Map Aggregate Functions

```sh
presto> select histogram(id) from (values 1,1,2,2,3,1) as t(id);
      _col0      
-----------------
 {1=3, 2=2, 3=1} 
(1 row)

Query 20211218_023955_00053_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
237ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select map_agg(id,name) from (values (1,'aa'),(2,'bb')) as t(id,name);
    _col0     
--------------
 {1=aa, 2=bb} 
(1 row)

Query 20211218_024416_00056_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select map_agg(id,name) from (values (1,'aa'),(2,'bb'),(1,'cc')) as t(id,name);
    _col0     
--------------
 {1=aa, 2=bb} 
(1 row)

Query 20211218_025347_00065_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
181ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select multimap_agg(id,name) from (values (1,'aa'),(2,'bb'),(1,'cc')) as t(id,name);
        _col0         
----------------------
 {1=[aa, cc], 2=[bb]} 
(1 row)

Query 20211218_025323_00064_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
320ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select map_union(mapp) from (values map(array[1,2],array['aa','bb']),map(array[3,4],array['cc','dd'])) as t(mapp);
          _col0           
--------------------------
 {1=aa, 2=bb, 3=cc, 4=dd} 
(1 row)

Query 20211218_024754_00058_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
156ms [0 rows, 0B] [0 rows/s, 0B/s]

# 在这两个 map 中有两个 key 都为 1，返回其对应的值的时候，是随机选择，可能是 'aa'，也可能是 'cc'
presto> select map_union(mapp) from (values map(array[1,2],array['aa','bb']),map(array[1,4],array['cc','dd'])) as t(mapp);
       _col0        
--------------------
 {1=aa, 2=bb, 4=dd} 
(1 row)

Query 20211218_024813_00060_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
235ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select map_union_sum(mapp) from (values map(array['aa','bb'],array[1,2]),map(array['aa','cc'],array[2,3])) as t(mapp);
       _col0        
--------------------
 {aa=3, bb=2, cc=3} 
(1 row)

Query 20211218_025053_00061_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
422ms [0 rows, 0B] [0 rows/s, 0B/s]

# null被处理成0
presto> select map_union_sum(mapp) from (values map(array['aa','bb'],array[1,2]),map(array['aa','cc'],array[2,null])) as t(mapp);
       _col0        
--------------------
 {aa=3, bb=2, cc=0} 
(1 row)

Query 20211218_025119_00062_kjjye, FINISHED, 1 node
Splits: 1 total, 1 done (100.00%)
428ms [0 rows, 0B] [0 rows/s, 0B/s]
```

### Approximate Aggregate Functions

TODO

### Statistical Aggregate Functions

TODO

### Classification Metrics Aggregate Functions

TODO

### Differential Entropy Functions

TODO

## 14 Window Functions

来自官网：[https://prestodb.io/docs/current/functions/window.html](https://prestodb.io/docs/current/functions/window.html)

原文含有示例

参考：[https://github.com/ZGG2016/mysql-reference-manual/tree/master/12%20Functions%20and%20Operators/12.21%20%E7%AA%97%E5%8F%A3%E5%87%BD%E6%95%B0-Window%20Functions](https://github.com/ZGG2016/mysql-reference-manual/tree/master/12%20Functions%20and%20Operators/12.21%20%E7%AA%97%E5%8F%A3%E5%87%BD%E6%95%B0-Window%20Functions)

## 15 Array Functions and Operators

来自官网：[https://prestodb.io/docs/current/functions/array.html](https://prestodb.io/docs/current/functions/array.html)

```sh
presto> select all_match(array[1,2,3],x->x>0);
 _col0 
-------
 true  
(1 row)

Query 20211218_034206_00071_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select any_match(array[1,2,3],x->x>2);
 _col0 
-------
 true  
(1 row)

presto> select none_match(array[1,2,3,4],x->x>5);
 _col0 
-------
 true  
(1 row)

Query 20211218_042736_00110_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
122ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select none_match(array[1,2,3,4],x->x>2);
 _col0 
-------
 false 
(1 row)

Query 20211218_042740_00111_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
143ms [0 rows, 0B] [0 rows/s, 0B/s]

Query 20211218_034349_00072_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
360ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_average(array[1,2,3]);
 _col0 
-------
   2.0 
(1 row)

Query 20211218_034425_00073_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_average(array[1,2,3,null]);
 _col0 
-------
   2.0 
(1 row)

Query 20211218_034432_00074_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_distinct(array[1,2,3,1]);
   _col0   
-----------
 [1, 2, 3] 
(1 row)

Query 20211218_034512_00075_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
180ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_duplicates (array[1,2,3,1]);
 _col0 
-------
 [1]   
(1 row)

Query 20211218_034526_00076_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_except(array[1,2,3],array[3,4,5]);
 _col0  
--------
 [1, 2] 
(1 row)

Query 20211218_034606_00077_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
228ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_except(array[1,2,1,3],array[3,4,5]);
 _col0  
--------
 [1, 2] 
(1 row)

Query 20211218_034711_00079_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
116ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_frequency(array[1,2,1,3]);
      _col0      
-----------------
 {1=2, 2=1, 3=1} 
(1 row)

Query 20211218_034804_00080_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
325ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_join(array['a','b',null],'-','str');
  _col0  
---------
 a-b-str 
(1 row)

Query 20211218_034954_00081_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
173ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_max(array[1,2,1,3]);
 _col0 
-------
     3 
(1 row)

Query 20211218_035647_00083_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
269ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_position(array['bb','aa','bb'],'bb');
 _col0 
-------
     1 
(1 row)

Query 20211218_035836_00087_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
129ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_position(array['bb','aa','bb'],'bb',2);
 _col0 
-------
     3 
(1 row)

Query 20211218_035917_00088_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
217ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_position(array['bb','aa','bb'],'bb',-2);
 _col0 
-------
     1 
(1 row)

Query 20211218_035930_00089_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
159ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_remove(array['bb','aa','bb'],'bb');
 _col0 
-------
 [aa]  
(1 row)

Query 20211218_040009_00090_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
188ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_sort(array['bb','aa','bb']);
    _col0     
--------------
 [aa, bb, bb] 
(1 row)

Query 20211218_040031_00091_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
208ms [0 rows, 0B] [0 rows/s, 0B/s]
```

```sql
SELECT array_sort(ARRAY [3, 2, 5, 1, 2], (x, y) -> IF(x < y, 1, IF(x = y, 0, -1))); -- [5, 3, 2, 2, 1]
SELECT array_sort(ARRAY ['bc', 'ab', 'dc'], (x, y) -> IF(x < y, 1, IF(x = y, 0, -1))); -- ['dc', 'bc', 'ab']
SELECT array_sort(ARRAY [3, 2, null, 5, null, 1, 2], -- sort null first with descending order
                  (x, y) -> CASE WHEN x IS NULL THEN -1
                                 WHEN y IS NULL THEN 1
                                 WHEN x < y THEN 1
                                 WHEN x = y THEN 0
                                 ELSE -1 END); -- [null, null, 5, 3, 2, 2, 1]
SELECT array_sort(ARRAY [3, 2, null, 5, null, 1, 2], -- sort null last with descending order
                  (x, y) -> CASE WHEN x IS NULL THEN 1
                                 WHEN y IS NULL THEN -1
                                 WHEN x < y THEN 1
                                 WHEN x = y THEN 0
                                 ELSE -1 END); -- [5, 3, 2, 2, 1, null, null]
SELECT array_sort(ARRAY ['a', 'abcd', 'abc'], -- sort by string length
                  (x, y) -> IF(length(x) < length(y),
                               -1,
                               IF(length(x) = length(y), 0, 1))); -- ['a', 'abc', 'abcd']
SELECT array_sort(ARRAY [ARRAY[2, 3, 1], ARRAY[4, 2, 1, 4], ARRAY[1, 2]], -- sort by array length
                  (x, y) -> IF(cardinality(x) < cardinality(y),
                               -1,
                               IF(cardinality(x) = cardinality(y), 0, 1))); -- [[1, 2], [2, 3, 1], [4, 2, 1, 4]]
```

```sh
presto> select array_sum(array[1,2,1,3]);
 _col0 
-------
     7 
(1 row)

Query 20211218_041207_00092_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
220ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_sum(array[null,null]);
 _col0 
-------
     0 
(1 row)

Query 20211218_041214_00093_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
243ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_sum(array[1,2,1,3,null]);
 _col0 
-------
     7 
(1 row)

Query 20211218_041221_00094_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
145ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select array_union(array[1,2,3],array[3,4,5]);
      _col0      
-----------------
 [1, 2, 3, 4, 5] 
(1 row)

Query 20211218_041620_00098_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
204ms [0 rows, 0B] [0 rows/s, 0B/s]

# 数组大小
presto> select cardinality(array[1,2,3]);
 _col0 
-------
     3 
(1 row)

Query 20211218_041647_00099_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
110ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select concat(array[1,2,3],array[3,4,5]);
       _col0        
--------------------
 [1, 2, 3, 3, 4, 5] 
(1 row)

Query 20211218_041837_00102_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
224ms [0 rows, 0B] [0 rows/s, 0B/s]

# 数组元素两两组合，第二个参数不能大于5
presto> SELECT combinations(ARRAY['foo', 'bar', 'boo'],2);
                _col0                 
--------------------------------------
 [[foo, bar], [foo, boo], [bar, boo]] 
(1 row)

Query 20211218_042021_00103_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
158ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select element_at(array['foo', 'bar', 'boo'],1);
 _col0 
-------
 foo   
(1 row)

Query 20211218_042207_00105_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
97ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> SELECT filter(ARRAY [5, -6, NULL, 7], x -> x > 0);
 _col0  
--------
 [5, 7] 
(1 row)

Query 20211218_042239_00106_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
145ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select flatten(array[array[1,2],array[3,4]]);
    _col0     
--------------
 [1, 2, 3, 4] 
(1 row)

Query 20211218_042350_00107_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
155ms [0 rows, 0B] [0 rows/s, 0B/s]

```

```sql
SELECT ngrams(ARRAY['foo', 'bar', 'baz', 'foo'], 2); -- [['foo', 'bar'], ['bar', 'baz'], ['baz', 'foo']]
SELECT ngrams(ARRAY['foo', 'bar', 'baz', 'foo'], 3); -- [['foo', 'bar', 'baz'], ['bar', 'baz', 'foo']]
SELECT ngrams(ARRAY['foo', 'bar', 'baz', 'foo'], 4); -- [['foo', 'bar', 'baz', 'foo']]
SELECT ngrams(ARRAY['foo', 'bar', 'baz', 'foo'], 5); -- [['foo', 'bar', 'baz', 'foo']]
SELECT ngrams(ARRAY[1, 2, 3, 4], 2); -- [[1, 2], [2, 3], [3, 4]]

SELECT reduce(ARRAY [], 0, (s, x) -> s + x, s -> s); -- 0
SELECT reduce(ARRAY [5, 20, 50], 0, (s, x) -> s + x, s -> s); -- 75
SELECT reduce(ARRAY [5, 20, NULL, 50], 0, (s, x) -> s + x, s -> s); -- NULL
SELECT reduce(ARRAY [5, 20, NULL, 50], 0, (s, x) -> s + COALESCE(x, 0), s -> s); -- 75
SELECT reduce(ARRAY [5, 20, NULL, 50], 0, (s, x) -> IF(x IS NULL, s, s + x), s -> s); -- 75
SELECT reduce(ARRAY [2147483647, 1], CAST (0 AS BIGINT), (s, x) -> s + x, s -> s); -- 2147483648
SELECT reduce(ARRAY [5, 6, 10, 20], -- calculates arithmetic average: 10.25
              CAST(ROW(0.0, 0) AS ROW(sum DOUBLE, count INTEGER)),
              (s, x) -> CAST(ROW(x + s.sum, s.count + 1) AS ROW(sum DOUBLE, count INTEGER)),
              s -> IF(s.count = 0, NULL, s.sum / s.count));
```

```sh
presto> select repeat(1,2);
 _col0  
--------
 [1, 1] 
(1 row)

Query 20211218_043222_00112_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
197ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select reverse(array[1,2,3,4]);
    _col0     
--------------
 [4, 3, 2, 1] 
(1 row)

Query 20211218_043245_00113_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
91ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select sequence(2,5);
    _col0     
--------------
 [2, 3, 4, 5] 
(1 row)

Query 20211218_043309_00114_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
108ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select sequence(DATE '2001-08-22',DATE '2011-08-22',INTERVAL '2' year TO month);
                                  _col0                                   
--------------------------------------------------------------------------
 [2001-08-22, 2003-08-22, 2005-08-22, 2007-08-22, 2009-08-22, 2011-08-22] 
(1 row)

Query 20211218_043914_00120_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
203ms [0 rows, 0B] [0 rows/s, 0B/s]


presto> select slice(array['aa','bb','cc'],1,2);
  _col0   
----------
 [aa, bb] 
(1 row)

Query 20211218_044013_00122_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
127ms [0 rows, 0B] [0 rows/s, 0B/s]

presto> select slice(array['aa','bb','cc'],-2,2);
  _col0   
----------
 [bb, cc] 
(1 row)

Query 20211218_044144_00127_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
122ms [0 rows, 0B] [0 rows/s, 0B/s]
```

```sql
SELECT transform(ARRAY [], x -> x + 1); -- []
SELECT transform(ARRAY [5, 6], x -> x + 1); -- [6, 7]
SELECT transform(ARRAY [5, NULL, 6], x -> COALESCE(x, 0) + 1); -- [6, 1, 7]
SELECT transform(ARRAY ['x', 'abc', 'z'], x -> x || '0'); -- ['x0', 'abc0', 'z0']
SELECT transform(ARRAY [ARRAY [1, NULL, 2], ARRAY[3, NULL]], a -> filter(a, x -> x IS NOT NULL)); -- [[1, 2], [3]]
```

```sh
presto> SELECT zip(ARRAY[1, 2], ARRAY['1b', null, '3b']);
                                   _col0                                    
----------------------------------------------------------------------------
 [{field0=1, field1=1b}, {field0=2, field1=null}, {field0=null, field1=3b}] 
(1 row)

Query 20211218_044245_00128_kjjye, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
452ms [0 rows, 0B] [0 rows/s, 0B/s]
```

```sql
-- 指定形式
SELECT zip_with(ARRAY[1, 3, 5], ARRAY['a', 'b', 'c'], (x, y) -> (y, x)); -- [ROW('a', 1), ROW('b', 3), ROW('c', 5)]
SELECT zip_with(ARRAY[1, 2], ARRAY[3, 4], (x, y) -> x + y); -- [4, 6]
SELECT zip_with(ARRAY['a', 'b', 'c'], ARRAY['d', 'e', 'f'], (x, y) -> concat(x, y)); -- ['ad', 'be', 'cf']
SELECT zip_with(ARRAY['a'], ARRAY['d', null, 'f'], (x, y) -> coalesce(x, y)); -- ['a', null, 'f']
```

## 16 Map Functions and Operators

来自官网：[https://prestodb.io/docs/current/functions/map.html](https://prestodb.io/docs/current/functions/map.html)

原文含示例

思路类似array

## 17 URL Functions

TODO

## 18 IP Functions

## 19 Geospatial Functions

## 20 HyperLogLog Functions

## 21 KHyperLogLog Functions

## 22 Quantile Digest Functions

## 23 Color Functions

## 24 Session Information

## 25 Teradata Functions

## 26 Internationalization Functions