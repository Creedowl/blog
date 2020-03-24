---
title: PostgreSQL 简介与常用命令
date: 2019-02-03T18:32:39+08:00
draft: false
toc: true
tags:
 - postgres
 - handbook
---
# PostgreSQL 简介与常用命令

PostgreSQL，简称postgres，是继mysql被收购之后极为热门的开源关系型数据库，这篇文章简单记录一些使用过程中的基本用法。

## 安装

为了保证宿主机环境的纯净，这里使用docker来安装postgres

```shell
docker run --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=password -d postgres
```

常用环境变量

> POSTGRES_USER: 数据库用户，缺省postgres
>
> POSTGRES_PASSWORD: 数据库密码 ps. 若从localhost连接则默认trust，不需要填写
>
> POSTGRES_DB: 数据库名，缺省postgres

更多docker相关内容参考[postgres官方docker镜像页面](https://hub.docker.com/_/postgres)

## 常用命令

**连接数据库**

```shell
psql -U user -d database -h host -p port
```

**导入数据**

```shell
psql database < db.sql
```

**控制台命令**

> \h：查看SQL命令的解释，比如\h select。
>
> \?：查看psql命令列表。
>
> \l：列出所有数据库。
>
> \c [database_name]：连接其他数据库。
>
> \d：列出当前数据库的所有表格。
>
> \d [table_name]：列出某一张表格的结构。
>
> \du：列出所有用户。
>
> \e：打开文本编辑器。
>
> \conninfo：列出当前数据库和连接的信息。

**数据库操作**

*采用基本sql语法*

```sql
# 建表
CREATE TABLE IF NOT EXISTS user (
	id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);

# 插入
INSERT INTO user (name) VALUES ('username');

# 选择
SELECT * FROM user;

# 更新
UPDATE user SET name = 'qwer' WHERE id = 1;

# 删除
DELETE FROM user WHERE id = 1;

# 添加栏位
ALTER TABLE user ADD gender VARCHAR(2);

# 更新表结构
ALTER TABLE user ALTER COLUMN gender SET NOT NULL;

# 字段更名
ALTER TABLE user RENAME COLUMN name TO username;

# 删除字段
ALTER TABLE user DROP COLUMN gender;

# 表名更改
ALTER TABLE user RENAME TO users;

# 删除表格
DROP TABLE users;
```

## 相关链接

> [PostgreSQL新手入门](http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html)

