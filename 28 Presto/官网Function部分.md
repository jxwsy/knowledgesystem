# 官网 Function 部分

版本：0.266

[TOC]

## 1 创建

### 语法

```sql
CREATE [ OR REPLACE ] [TEMPORARY] FUNCTION
qualified_function_name (
  parameter_name parameter_type
  [, ...]
)
RETURNS return_type
[ COMMENT function_description ]
[ LANGUAGE [ SQL | identifier] ]
[ DETERMINISTIC | NOT DETERMINISTIC ]
[ RETURNS NULL ON NULL INPUT | CALLED ON NULL INPUT ]
[ RETURN expression | EXTERNAL [ NAME identifier ] ]
```

- `OR REPLACE` 

	如果已存在一个函数名称和参数类型列表都一样的函数，那么指定 `OR REPLACE` 就会覆盖已存在的函数而创建一个新的函数。

- `TEMPORARY`

	指定 `TEMPORARY` 创建的表只会在当前的 session 里有效，没有持久化项。

	每个临时函数由函数名称唯一确定。函数名称不能被限定，如 `hive.default.tan`，且不能和已存在的内建函数名称一样。

	而持久化函数由限定的函数名称和参数类型列表唯一确定。

- qualified_function_name 

	qualified_function_name 需要是 `catalog.schema.function_name` 格式。

	要创建一个持久化函数，其所在的函数名称空间（以 `catalog.schema`） 格式）必须被一个函数名称空间管理器管理(Function Namespace Managers)（如何创建具体见第二节）。

	临时函数可以直接创建，持久化函数需要先配置函数名称空间管理器。

- return_type 

	返回类型需要和程序主体(routine body) expression 的类型相匹配上，且不需要进行转换。


- 程序特征

	每种程序特征最多指定一次

程序特征  |  默认值  | 描述
---|:---|:---
Language clause | SQL  |  The language in which the function is defined.
Deterministic characteristic |  NOT DETERMINISTIC | Whether the function is deterministic. NOT DETERMINISTIC means that the function is possibly non-deterministic.
Null-call clause |  CALLED ON NULL INPUT【？？】 | The behavior of the function in which null is supplied as the value of at least one argument.

[DETERMINISTIC](https://blog.csdn.net/java3344520/article/details/7647487):它表示一个函数在输入不变的情况下输出是否确定。如果你的函数当输入一样时,会返回同样的结果.

**【identifier 、EXTERNAL [ NAME identifier ]  ？？】**


### 函数名称空间管理器(Function Namespace Managers)

要创建持久化函数，需要先配置一个函数名称空间管理器。

函数名称空间不支持存储表和视图，仅支持函数。

内建函数都在一个名为 `presto.default` 的名称空间中。

每个名称空间都绑定了一个 catalog 名字，在这个 catalog 中管理所有的函数，但它不是真正的 catalog。

具体描述见官网原文。

以下是配置过程：

启动 mysql 服务，并在 mysql 中，创建相关的数据库。（当前仅支持mysql）

```sql
mysql> CREATE DATABASE presto;
Query OK, 1 row affected (0.12 sec)

mysql> USE presto;
Database changed
``` 

在 presto 中新建配置文件，并分发到其他节点。

```sh
[root@bigdata101 function-namespace]# pwd
/opt/presto-0.266/etc/function-namespace
[root@bigdata101 function-namespace]# ls
example.properties

# 字段含义见官网原文
[root@bigdata101 function-namespace]# cat example.properties 
function-namespace-manager.name=mysql
database-url=jdbc:mysql://bigdata101:3306/presto
function-namespaces-table-name=func_namespaces
functions-table-name=functions   
```

重启 presto 服务。重启后会自动创建如下表：

```sql
mysql> show tables;
+--------------------+
| Tables_in_presto   |
+--------------------+
| func_namespaces    |
| functions          |
| user_defined_types |
+--------------------+
3 rows in set (0.37 sec)

mysql> select * from func_namespaces;
Empty set (0.14 sec)
```

要创建一个新的函数名称空间，往 func_namespaces 表里插入数据：

```sql
mysql> INSERT INTO func_namespaces (catalog_name, schema_name)
    -> VALUES('example', 'default');
Query OK, 1 row affected (0.04 sec)

mysql> select * from func_namespaces;
+--------------+-------------+
| catalog_name | schema_name |
+--------------+-------------+
| example      | default     |
+--------------+-------------+
1 row in set (0.00 sec)
```

在 presto 中，新建一个函数

```sh
presto> CREATE FUNCTION example.default.tan_test(x double) RETURNS double DETERMINISTIC RETURNS NULL ON NULL INPUT RETURN sin(x) / cos(x);
CREATE FUNCTION
```

在 mysql 中查看

```sh
mysql> select * from functions;
+----+------------------------------------------------------------------+----------------------------------+---------+--------------+-------------+---------------+---------------------------------------------+-------------+----------------------------------------------------------------------------------------------------------------+------------------------------+-------------+---------+-------------+---------------------+---------------------+
| id | function_id_hash                                                 | function_id                      | version | catalog_name | schema_name | function_name | parameters                                  | return_type | routine_characteristics                                                                                        | body                         | description | deleted | delete_time | create_time         | update_time         |
+----+------------------------------------------------------------------+----------------------------------+---------+--------------+-------------+---------------+---------------------------------------------+-------------+----------------------------------------------------------------------------------------------------------------+------------------------------+-------------+---------+-------------+---------------------+---------------------+
|  1 | 4371899fc2d8398aab65b717043d5a3b83b3136c49e51fe8af002a7b7e2cdc3b | example.default.tan_test(double) |       1 | example      | default     | tan_test      | [ {
  "name" : "x",
  "type" : "double"
} ] | double      | {
  "language" : "SQL",
  "determinism" : "DETERMINISTIC",
  "nullCallClause" : "RETURNS_NULL_ON_NULL_INPUT"
} | RETURN ("sin"(x) / "cos"(x)) |             |       0 | NULL        | 2021-12-18 23:53:13 | 2021-12-18 23:53:13 |
+----+------------------------------------------------------------------+----------------------------------+---------+--------------+-------------+---------------+---------------------------------------------+-------------+----------------------------------------------------------------------------------------------------------------+------------------------------+-------------+---------+-------------+---------------------+---------------------+
1 row in set (0.01 sec)
```

在配置过程中，如果服务自动关闭，日志中出现 `access deny` 错误，可以在 my.cnf 中添加 `skip-grant-tables` 配置，再重启 mysql 服务。


使用这个函数：

```sh
presto> select example.default.tan_test(45);
       _col0        
--------------------
 1.6197751905438615 
(1 row)

Query 20211218_155831_00002_k2y7j, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:03 [0 rows, 0B] [0 rows/s, 0B/s]

presto> select example.default.tan_test(null);
 _col0 
-------
 NULL  
(1 row)

Query 20211218_155839_00003_k2y7j, FINISHED, 1 node
Splits: 17 total, 17 done (100.00%)
0:01 [0 rows, 0B] [0 rows/s, 0B/s]


presto> CREATE TEMPORARY FUNCTION square_test(x int)
     -> RETURNS int
     -> RETURN x * x;
CREATE FUNCTION
```

## 2 查看

    SHOW FUNCTIONS [ LIKE pattern [ ESCAPE 'escape_character' ] ]

```sh
presto> show functions;
             Function             |           Return Type            |                                 Argume
----------------------------------+----------------------------------+---------------------------------------
 abs                              | bigint                           | bigint                                
 abs                              | decimal(p,s)                     | decimal(p,s)                          
 abs                              | double                           | double                                
 abs                              | integer                          | integer                               
 abs                              | real                             | real                                  
 abs                              | smallint                         | smallint                              
 abs                              | tinyint                          | tinyint                               
 acos                             | double                           | double                                
 all_match                        | boolean                          | array(T), function(T,boolean)         
 any_match                        | boolean                          | array(T), function(T,boolean) 
...

# 没有搜到上面创建的 tan_test  ????
presto> show functions like 'str%';
 Function | Return Type |         Argument Types         | Function Type | Deterministic |                   
----------+-------------+--------------------------------+---------------+---------------+-------------------
 strpos   | bigint      | varchar(x), varchar(y)         | scalar        | true          | returns index of f
 strpos   | bigint      | varchar(x), varchar(y), bigint | scalar        | true          | returns index of n
 strrpos  | bigint      | varchar(x), varchar(y)         | scalar        | true          | returns index of l
 strrpos  | bigint      | varchar(x), varchar(y), bigint | scalar        | true          | returns index of n
(4 rows)
(END)
...

presto> show functions like 'st|_%' ESCAPE '|';
      Function       |    Return Type     |             Argument Types             | Function Type | Determin
---------------------+--------------------+----------------------------------------+---------------+---------
 st_area             | double             | Geometry                               | scalar        | true    
 st_area             | double             | SphericalGeography                     | scalar        | true    
 st_asbinary         | varbinary          | Geometry                               | scalar        | true    
 st_astext           | varchar            | Geometry                               | scalar        | true    
 st_astext           | varchar            | SphericalGeography                     | scalar        | true    
```

## 3 修改

```sql
ALTER FUNCTION qualified_function_name [ ( parameter_type[, ...] ) ]
RETURNS NULL ON NULL INPUT | CALLED ON NULL INPUT
```

如果含重名的函数，则必须要带 parameter_type。如果没有，则不必要。

当前仅支持修改 null-call 子句。

```sql
presto> ALTER FUNCTION example.default.tan_test(double)
     -> CALLED ON NULL INPUT;
ALTER FUNCTION
```

## 4 删除

```sql
DROP [TEMPORARY] FUNCTION [ IF EXISTS ] qualified_function_name [ ( parameter_type[, ...] ) ]
```

可以指定可选 parameter_type 来将匹配缩小到特定的函数。

```sh
# 只匹配到一个函数的时候，可以直接这样删除。如果存在函数名相同的情况，需要指定参数。
presto> drop FUNCTION example.default.tan_test;
DROP FUNCTION

presto> DROP FUNCTION example.default.tan_test(double)
```

--------------------------------------------------------

来自官网：

[https://prestodb.io/docs/0.266/sql/create-function.html](https://prestodb.io/docs/0.266/sql/create-function.html)

[https://prestodb.io/docs/0.266/admin/function-namespace-managers.html](https://prestodb.io/docs/0.266/admin/function-namespace-managers.html)

[https://prestodb.io/docs/0.266/sql/show-functions.html](https://prestodb.io/docs/0.266/sql/show-functions.html)

[https://prestodb.io/docs/0.266/sql/alter-function.html](https://prestodb.io/docs/0.266/sql/alter-function.html)

[https://prestodb.io/docs/0.266/sql/drop-function.html](https://prestodb.io/docs/0.266/sql/drop-function.html)