---
title: mysql笔记
date: 2023-02-07 21:09:17
tags: "数据库"
categories: "mysql"
---

#### 常用命令

##### mysql启动停止命令

```
启动：net start [mysql服务名]
停止：net stop [mysql服务名]
```

##### mysql连接

`mysql [-h 127.0.0.1] [-p 3306] -u root -p`

#### SQL通用语法

- 可以单行或多行书写，以分号结尾
- 可以使用若干空格/缩进增强语句的可读性
- mysql数据库的SQL语句不区分大小写，关键字建议使用大写
- 单行注释：`-- 注释内容`或`# 注释内容`
- 多行注释：`/* 注释内容 */`

#### SQL分类

- DDL（Data Definition Language）：数据定义语言，用于定义数据库对象（数据库、表、字段）
- DML（Data Manipulation Language）：数据操作语言，用于对数据库中的数据进行增删改
- DQL（Data Query Language）：数据查询语言，用于查询数据库表的记录
- DCL（Data Control Language）：数据控制语言，用于创建数据库用户、控制数据库的访问权限

#### DDL

##### DDL数据库操作

###### 查询

- 查询所有数据库：`show database;`
- 查询当前数据库：`select database();`

###### 创建

`create database [if not exists] 数据库名 [default charset 字符集] [collate 排序规则];`

###### 删除

`drop database [if exists] 数据库名;`

###### 使用

`use 数据库名`

##### DDL表操作

MySQL数据类型主要分为3类：数值类型、字符串类型、日期时间类型：

| 数值类型     | 大小  | 有符号范围     | 无符号范围     |
| ------------ | ----- | -------------- | -------------- |
| tinyint      | 1byte | (-128,127)     | (0,255)        |
| smallint     | 2byte | (-32768,32767) | (0,65535)      |
| int或integer | 4byte | (±2147483648)  | (0,4294967295) |
| bigint       | 8byte | (-2^63,2^63-1) | (0,2^64-1)     |
| float        | 4byte |                |                |
| double       | 8byte |                |                |

| 字符串类型 | 大小        | 描述                        |
| ---------- | ----------- | --------------------------- |
| char       | 0-255       | 定长字符串（性能高）        |
| varchar    | 0-65535     | 变长字符串（性能较差）      |
| tinyblob   | 0-255       | 不超过255个字节的二进制数据 |
| tinytext   | 0-255       | 短文本字符串                |
| blob       | 0-65535     | 二进制长文本数据            |
| text       | 0-65535     | 长文本数据                  |
| longblob   | 0-429467295 | 二进制极大文本数据          |
| longtext   | 0-429467295 | 极大长文本数据              |

| 日期时间类型 | 大小 | 范围                                     | 格式                |
| ------------ | ---- | ---------------------------------------- | ------------------- |
| date         | 3    | 1000-01-01至9999-12-31                   | YYYY-MM-DD          |
| time         | 3    | -838:59:59至838:59:59                    | HH:MM:SS            |
| year         | 1    | 1901至2155                               | YYYY                |
| datetime     | 8    | 1000-01-01 00:00:00至9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS |

###### 查询

- 查询当前数据库所有表：`show tables;`
- 查询表结构：`desc 表名;`
- 查询指定表的建表语句：`show create table 表名;`

###### 创建

```
create table 表名(
        字段1 字段1的类型 [comment '字段1注释']，
        字段2 字段2的类型 [comment '字段2注释']，
        字段3 字段3的类型 [comment '字段3注释']，
        ......
        字段n 字段n的类型 [comment '字段n注释']
)[comment '表注释'];
```

###### 修改

- 添加字段：`alter table 表名 add 字段名 类型(长度) [comment 注释] [约束];`
- 修改数据类型：`alter table 表名 modify 字段名 新数据类型(长度);`
- 修改字段名和字段类型：`alter table 表名 change 旧字段名 新字段名 类型(长度) [comment 注释] [约束];`
- 删除字段：`alter table 表名 drop 字段名;`
- 修改表名：`alter table 表名 rename to 新表名;`

###### 删除

- 删除表：`drop table [if exists] 表名;`
- 删除指定表，并重新创建该表：`truncate table 表名;`
