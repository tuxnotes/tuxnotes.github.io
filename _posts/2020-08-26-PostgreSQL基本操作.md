---
layout: post
title: PostgreSQL基本操作
date: 2020-08-26
author: tux
tags: postgresql
---

# 连接

```bash
psql -d db1 -U userA
```

# 执行sql脚本

方法一：连接之后

```bash
\i /path/to/script.sql
```

方法二：直接在系统的命令行

```bash
psql -d db1 -U userA -f /pathA/xxx.sql
```

# 执行系统命令

```bash
\! ls
```

# 切换数据库

```bash
\c DBNAME
```

# 创建数据库，用户，授权

```bash
CREATE DATABASE yourdbname;
CREATE USER youruser WITH ENCRYPTED PASSWORD 'yourpass';
GRANT ALL PRIVILEGES ON DATABASE yourdbname TO youruser;
```

# 改变数据库的owner

```bash
ALTER DATABASE name OWNER TO new_owner;
```

# 查看当前数据库的所有表

方法一：通过命令查询

```bash
\d DBNAME # 得到所有表的名字 
\d TABLE_NAME # 得到表结构
```

方法二：通过SQL语句查询

```sql
select * from pg_tables; # 得到当前db中所有表的信息（这里pg_tables是系统视图）
select tablename from pg_tables where schemaname='public' # 得到所有用户自定义表的名字（这里"tablename"字段是表的名字，"schemaname"是schema的名字。用户自定义的表，如果未经特殊处理，默认都是放在名为public的schema下）
```
