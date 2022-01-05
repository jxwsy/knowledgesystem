# superset连接oracle

[TOC]

## 1 操作过程

1. 安装驱动

```
conda install cx_Oracle
```

2. 安装 oracle 客户端(instantclient)


3. 将 instantclient 目录下的 dll 文件复制到 miniconda 目录下，见下图：

![连接oracle](./image/连接oracle.png)

4. 重启 superset 服务

5. 在 oracle 中创建新用户，并赋予权限

```sql
CREATE TABLESPACE TEST
LOGGING
DATAFILE 'D:\ORACLE_TABLESPACE\TEST\TEST.DBF'
SIZE 100M
AUTOEXTEND ON
NEXT 100M MAXSIZE UNLIMITED
EXTENT MANAGEMENT LOCAL;

CREATE TEMPORARY TABLESPACE TEST_TEMP
TEMPFILE 'D:\ORACLE_TABLESPACE\TEST_TEMP\TEST_TEMP.DBF'
SIZE 100M
AUTOEXTEND ON
NEXT 100M MAXSIZE UNLIMITED
EXTENT MANAGEMENT LOCAL;

create user testuser identified by testuser2021 
  default tablespace TEST 
  temporary tablespace TEST_TEMP;  

grant connect,resource,dba to testuser;
```

6. superset 连接 oracle

```
oracle://testuser:testuser2021@<hostname>:<port>/<databasename>
```

## 2 问题

### 2.1 问题1

`ERROR: (cx_Oracle.DatabaseError) DPI-1047: Cannot locate a 64-bit Oracle Client library: "The specified module could not be found". See https://cx-oracle.readthedocs.io/en/latest/user_guide/installation.html for help(Background on this error at: http://sqlalche.me/e/13/4xp6)`

原因是没有把 dll 文件复制到 miniconda 目录下。

### 2.2 问题2

`ERROR: (cx_Oracle.DatabaseError) ORA-12505: TNS: 监听程序当前无法识别连接描述符中所给出的 SID
(Background on this error at: http://sqlalche.me/e/13/4xp6)`

原因 oracle 客户端的 tnsnames.ora 配置文件配置错误。