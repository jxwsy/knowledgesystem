
```sql
MERGE INTO transactions AS T 
USING merge_source AS S
ON T.ID = S.ID and T.tran_date = S.tran_date
WHEN MATCHED AND (T.TranValue != S.TranValue AND S.TranValue IS NOT NULL) THEN UPDATE SET TranValue = S.TranValue, last_update_user = 'merge_update'
WHEN MATCHED AND S.TranValue IS NULL THEN DELETE
WHEN NOT MATCHED THEN INSERT VALUES (S.ID, S.TranValue, 'merge_insert', S.tran_date);
```

```python

```
```sql

```


<font face="等线" color="grey">Apache Griffin measure module needs two configuration files to define the parameters of execution, one is for environment, the other is for dq job.</font>

<font face="微软雅黑" color="black">Apache Griffin 度量模块需要两个配置文件来定义执行参数，一个是对执行环境的配置文件，一个是对 dq job 的配置文件。 </font>