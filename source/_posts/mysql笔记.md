---
title: mysql笔记
date: 2023-02-07 21:09:17
tags: "mysql"
categories: "数据库"
---

## 基础篇

### 常用命令

#### mysql启动停止命令

```
启动：net start [mysql服务名]
停止：net stop [mysql服务名]
```

#### mysql连接

`mysql [-h 127.0.0.1] [-p 3306] -u root -p`

### SQL通用语法

- 可以单行或多行书写，以分号结尾
- 可以使用若干空格/缩进增强语句的可读性
- mysql数据库的SQL语句不区分大小写，关键字建议使用大写
- 单行注释：`-- 注释内容`或`# 注释内容`
- 多行注释：`/* 注释内容 */`

### MySQL语言

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

#### DML

##### 添加数据

- 添加指定字段：`insert into 表名 (字段名1, ...) values (值1, ...);`
- 添加全部字段：`insert into values (值1,...);`
- 批量添加数据：`insert into 表名 (字段名1, 字段名2, ...) VALUES (值1, 值2, ...), (值1, 值2, ...), (值1, 值2, ...);`

**注：**字符串和日期类型数据应包含在引号中；插入数据大小应在字段的规定范围内

##### 更新和删除数据

- 修改数据：`update 表名 set 字段名1 = 值1, 字段名2 = 值2, ... [where 条件];`
- 删除数据：`delete from 表名 [where 条件];`

#### DQL

```mysql
SELECT
	字段列表
FROM
	表名字段
WHERE
	条件列表
GROUP BY
	分组字段列表
HAVING
	分组后条件列表
ORDER BY
	排序字段列表
LIMIT
	分页参数
```

- 去除重复记录：`select distinct 字段列表 from 表名;`
- 转义：`select * from 表名 where name like '/_张三' escape '/';`/之后的_不作为通配符

- 条件查询条件列表：

  | 比较运算符          | 功能                                        |
  | ------------------- | ------------------------------------------- |
  | >                   | 大于                                        |
  | >=                  | 大于等于                                    |
  | <                   | 小于                                        |
  | <=                  | 小于等于                                    |
  | =                   | 等于                                        |
  | <> 或 !=            | 不等于                                      |
  | BETWEEN ... AND ... | 在某个范围内（含最小、最大值）              |
  | IN(...)             | 在in之后的列表中的值，多选一                |
  | LIKE 占位符         | 模糊匹配（\_匹配单个字符，%匹配任意个字符） |
  | IS NULL             | 是NULL                                      |

  | 逻辑运算符         | 功能                         |
  | ------------------ | ---------------------------- |
  | AND 或 &&          | 并且（多个条件同时成立）     |
  | OR 或 &#124;&#124; | 或者（多个条件任意一个成立） |
  | NOT 或 !           | 非，不是                     |

- 聚合查询常见聚合函数：

  | 函数  | 功能     |
  | ----- | -------- |
  | count | 统计数量 |
  | max   | 最大值   |
  | min   | 最小值   |
  | avg   | 平均值   |
  | sum   | 求和     |

- 分组查询：

  ```
  where和having的区别：
  	执行时机不同：where是分组前过滤，不满足条件的不参与分组；having是分组后对结果进行过滤
  	判断条件不同：where不能对聚合函数进行判断，而having可以
  	
  执行顺序：where > 聚合函数 > having
  分组之后查询字段一般为聚合函数和分组函数，查询其它字段无任何意义
  ```

- 排序查询：`select 字段列表 from 表名 order by 字段1 排序方式1, 字段2 排序方式2;`

  ASC：升序、DESC：降序

  若为多字段排序，只有当第一个字段值相同时才会根据第二个字段进行排序

- 分页查询：`select 字段列表 from 表名 limit 起始索引, 查询记录数;`

  起始索引从0开始，起始索引 = （查询页码 - 1） * 每页显示记录数；

  分页查询是数据库的方言，不同数据库有不同实现，MySQL是LIMIT；

  如果查询的是第一页数据，起始索引可以省略，直接简写 LIMIT 10；

- DQL执行顺序：FROM -> WHERE -> GROUP BY -> SELECT -> ORDER BY -> LIMIT

#### DCL

##### 查询用户

`select * from user;`

##### 创建用户

`create user '用户名'@'主机名' identified by '密码';`

##### 修改用户密码

`alter user '用户名'@'主机名' identified with mysql_native_password by '新密码';`

##### 删除用户

`drop user '用户名'@'主机名';`

```mysql
-- 创建用户test，只能在当前主机localhost访问
create user 'test'@'localhost' identified by '123456';
-- 创建用户test，能在任意主机访问
create user 'test'@'%' identified by '123456';
create user 'test' identified by '123456';
-- 修改密码
alter user 'test'@'localhost' identified with mysql_native_password by '1234';
-- 删除用户
drop user 'test'@'localhost';
```

常用权限：

| 权限                | 说明               |
| ------------------- | ------------------ |
| ALL, ALL PRIVILEGES | 所有权限           |
| SELECT              | 查询数据           |
| INSERT              | 插入数据           |
| UPDATE              | 修改数据           |
| DELETE              | 删除数据           |
| ALTER               | 修改表             |
| DROP                | 删除数据库/表/视图 |
| CREATE              | 创建数据库/表      |

- 查询权限：`show grants for '用户名'@'主机名';`
- 授予权限：`grant 权限列表 on 数据库名.表名 to '用户名'@'主机名';`
- 撤销权限：`revoke 权限列表 on 数据库名.表名 from '用户名'@'主机名';`

**注意事项**

- 多个权限用逗号分隔
- 授权时，数据库名和表名可以用*进行通配，代表所有

### MySQL函数

#### 字符串函数

| 函数                             | 功能                                                      |
| -------------------------------- | --------------------------------------------------------- |
| CONCAT(s1, s2, ..., sn)          | 字符串拼接，将s1, s2, ..., sn拼接成一个字符串             |
| LOWER(str)                       | 将字符串全部转为小写                                      |
| UPPER(str)                       | 将字符串全部转为大写                                      |
| LPAD(str, n, pad)                | 左填充，用字符串pad对str的左边进行填充，达到n个字符串长度 |
| RPAD(str, n, pad)                | 右填充，用字符串pad对str的右边进行填充，达到n个字符串长度 |
| TRIM(str)                        | 去掉字符串头部和尾部的空格                                |
| SUBSTRING(str, start, len)       | 返回从字符串str从start位置起的len个长度的字符串           |
| REPLACE(column, source, replace) | 替换字符串                                                |

#### 数值函数

| 函数       | 功能                             |
| ---------- | -------------------------------- |
| CEIL(x)    | 向上取整                         |
| FLOOR(x)   | 向下取整                         |
| MOD(x)     | 返回x/y的模                      |
| RAND()     | 返回0~1内的随机数                |
| ROUND(x,y) | 求参数x的四舍五入值，保留y位小数 |

#### 日期函数

| 函数                              | 功能                                        |
| --------------------------------- | ------------------------------------------- |
| CURDATE()                         | 返回当前日期                                |
| CURTIME()                         | 返回当前时间                                |
| NOW()                             | 返回当前日期和时间                          |
| YEAR(date)                        | 获取指定date的年份                          |
| MONTH(date)                       | 获取指定date的月份                          |
| DAY(date)                         | 获取指定date的日期                          |
| DATE_ADD(date,INTERVAL expr type) | 返回一个日期/时间加上时间间隔expr后的时间值 |
| DATEDIFF(date1,date2)             | 返回起始时间date1和结束时间date2之间的天数  |

#### 流程函数

| 函数                                                       | 功能                                               |
| ---------------------------------------------------------- | -------------------------------------------------- |
| IF(value,t,f)                                              | 若value为true则返回t，否则返回f                    |
| IFNULL(value1,value2)                                      | 若value1不为空，返回value1，否则返回value2         |
| CASE WHEN [val1] THEN [res1] ... ELSE [default] END        | 若val1为true，返回res1，...否则返回default默认值   |
| CASE [expr] WHEN [val1] THEN [res1] ... ELSE [default] END | 若expr值为val1，返回res1，...否则返回default默认值 |

```mysql
select
	name,(case when age > 30 then '中年' else '青年' end)
from employee;
```

### 约束

| 约束                    | 描述                                                     | 关键字         |
| ----------------------- | -------------------------------------------------------- | -------------- |
| 非空约束                | 限制该字段数据不能为null                                 | NOT NULL       |
| 唯一约束                | 保证字段所有数据都是唯一的                               | UNIQUE         |
| 主键约束                | 主键是一行数据的唯一标识，要求非空且唯一                 | PRIMARY KEY    |
|                         |                                                          | AUTO_INCREMENT |
| 默认约束                | 保存数据时，若未指定该字段的值，则采用默认值             | DEFAULT        |
| 检查约束（8.0.1版本后） | 保证字段值满足某一个条件                                 | CHECK          |
| 外键约束                | 用来让两张图的数据之间建立连接，保证数据的一致性和完整性 | FOREIGN KEY    |

```mysql
create table user (
	id int primary key auto_increment,
    name varchar(10) not null unique,
    age int check(age > 0 and age < 120),
    status char(1) default '1',
    gender char(1)
);
```

#### 外键约束

```mysql
# 添加外键
alter table emp add constraint fk_emp_dept_id foreign key(dept_id) references dept(id);

# 删除外键
alter table 表名 drop foreign key 外键名
```

### 多表查询

#### 多表关系

##### 一对多

```
案例：部门与员工
关系：一个部门对应多个员工，一个员工对应一个部门
实现：在多的一方建立外键，指向一的一方的主键
```

##### 多对多

```
案例：学生与课程
关系：一个学生可以选多门课程，一门课程可以供多个学生选修
实现：建立中间表，中间表至少包含两个外键，分别关联两方主键
```

##### 一对一

```
案例：用户与用户详情
关系：一对一关系，多用于单表拆分，基础字段和详情字段分别放在两张表，以提升操作效率
实现：在任意一方加入外键，关联另外一方的主键，并且设置外键为唯一的(UNIQUE)
```

#### 查询

##### 合并查询（笛卡尔积，展示所有组合结果）

> 笛卡尔积：两集合所有组合情况（在多表查询时需要消除无效的笛卡尔积）

`select * from employee,dept;`

- 消除无效笛卡尔积：`select * from employee,dept where employee.dept = dept.id;`

##### 内连接查询

内连接查询的是两张表交集的部分；显示内连接比隐式性能更高

- 隐式内连接：`select 字段列表 from 表1,表2 where 条件 ...;`
- 显示内连接：`select 字段列表 from 表1 [ inner ] join 表2 on 连接条件 ...;`

```mysql
-- 查询员工姓名及关联部门的名称
-- 隐式
select e.name,d.name from employee as e,dept as d where e.dept = d.id;
-- 显示
select e.name,d.name from employee as e inner join dept as d on e.dept = d.id;
```

##### 外连接查询

- 左外连接：查询左表所有数据和两张表交集数据

  `select 字段列表 from 表1 left [ outer ] join 表2 on 条件 ...; `

- 右外连接：查询右表所有数据和两张表交集数据

  `select 字段列表 from 表1 right [ outer ] join 表2 on 条件 ...;`

```mysql
-- 左
select e.*, d.name from employee as e left outer join dept as d on e.dept = d.id;
-- 右
select d.name, e.* from employee as e right outer join dept as d on e.dept = d.id;

# 左连接可以查询到没有dept的employee，右连接可以查询到没有employee的dept
```

##### 自连接查询

当前表与自身的连接查询，自连接必须使用表别名；自连接查询可以是内连接查询也可以是外连接查询

`select 字段列表 from 表A 别名A join 表A 别名B on 条件 ...;`

```mysql
-- 查询员工及所属领导的名字
select a.name,b.name from employee a,employee b where a.manager = b.id;
-- 没有领导的也查询出来
select a.name,b.name from employee a left join employee b on a.manager = b.id;
```

##### 联合查询union、union all

将多次查询的结果合并，形成一个新的查询集

**注：**union all会有重复结果，union不会；联合查询比使用or效率高，不会使索引失效

```mysql
select 字段列表 from 表A ...
union [all]
select 字段列表 from 表B ...
```

##### 子查询（嵌套查询）

子查询的位置可在`where之后`、`from之后`、`select之后`

###### 列子查询

返回的结果是一列（可以是多行）

- 常用操作符：in、not in、any、some（等同于any）、all

```mysql
-- 查询销售部和市场部所有员工信息
select * from employee where dept in (select id from dept where name = '销售部' or name = '市场部')
-- 查询比财务部所有人工资都高的员工信息
select * from employee where salary > all(select salary from employee where dept = (select id from dept where name = '财务部'));
-- 查询比研发部任意一人工资高的员工信息
select * from employee where salary > any(select salary from employ where dept = (select id from dept where name = '研发部'))；
```

###### 行子查询

返回的结果是一行（可以是多列）

- 常用操作符：=、<、>、in、not in

```mysql
-- 查询与xxx的薪资及直属领导相同的员工信息
select * from employee where (salary,manager) = (12500,1);
select * from employee where (salary,manager) = (select salary,manager from employee where name = 'xxx');
```

###### 表子查询

返回的结果是多行多列

- 常用操作符：in

```mysql
-- 查询与xxx1、xxx2的职位和薪资相同的员工
select * from employee where (job,salary) in (select job,salary from employee where name = 'xxx1' or name = 'xxx2');
-- 查询入职日期是2006-01-01之后的员工及部门信息
select e.*, d.* from (select * from employee where entrydate > '2006-01-01') as e left join dept as d on e.dept = d.id;
```

### 事务

一组操作的集合，事务将所有操作作为一个整体一起向系统提交或撤销操作请求；即这些操作要么同时成功，要么同时失败

```mysql
# 一、
-- 查看事务提交方式
select @@AUTOCOMMIT;
-- 设置事务提交方式，1为自动提交，0为手动提交，该设置只对当前会话有效
set @@AUTOCOMMIT = 0;
-- 提交事务
commit;
-- 回滚事务
rollback;

-- 事务实例（设置手动提交后）
select * from account where name = '张三';
update account set money = money - 1000 where name = '张三';
update account set money = money + 1000 where name = '李四';
commit;

# 二、
-- 开启事务
start transaction 或 begin transaction;
-- 提交事务
commit;
-- 回滚事务
rollback;

start transaction;
select * from account where name = '张三';
update account set money = money - 1000 where name = '张三';
update account set money = money + 1000 where name = '李四';
commit;
```

#### 并发事务

| 问题       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| 脏读       | 一个事务读到另一个事务还未提交的数据                         |
| 不可重复读 | 一个事务先后读取同一条记录，但两次读取的数据不同             |
| 幻读       | 一个事务按照条件查询数据时，没有对应的数据行，但是再插入数据时发现数据已经存在 |

##### 并发事务隔离级别

| 隔离级别              | 脏读 | 不可重复读 | 幻读 |
| --------------------- | ---- | ---------- | ---- |
| Read uncommitted      | √    | √          | √    |
| Read committed        | ×    | √          | √    |
| Repeatable Read(默认) | ×    | ×          | √    |
| Serializable          | ×    | ×          | ×    |

- √表示在当前隔离级别下该问题会出现
- Serializable 性能最低；Read uncommitted 性能最高，数据安全性最差

###### 查看事务隔离级别

`select @@TRANSACTION_ISOLATION;`

###### 设置事务隔离级别

`set [ SESSION | GLOBAL ] TRANSACTION ISOLATION  LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE };`

SESSION是会话级别，表示只针对当前会话有效，GLOBAL表示对所有会话有效

#### 四大特性ACID

- 原子性(Atomicity)：事务是不可分割的最小操作
- 一致性(Consistency)：事务完成时，必须使所有数据
- 隔离性(Isolation)：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行
- 持久性(Durability)：事务一旦提交或回滚，它对数据库数据的改变即永久的

### 三大范式

#### 第一范式（1NF）

原子性：保证每一列不可再分

#### 第二范式（2NF）

在满足第一范式的前提下实现每张表只描述一件事

#### 第三范式（3NF）

满足第一和第二范式的前提下实现确保数据表中每一列数据都和主键直接相关

#### 反范式适用场景

反范式通过增加冗余字段实现空间换时间从而提高查询的效率，但这会使得：

- 存储空间变大
- 一个表中字段做了修改，另一表中冗余字段也要同步修改，不然会导致数据不一致
- 若存储过程支持数据的更新、删除等操作。操作频繁会消耗系统资源
- 在数据量小的情况下反范式不能体现性能的优势

## 进阶篇

### 存储引擎

![](https://s1.ax1x.com/2023/02/13/pSoKwJH.png)

![](https://s1.ax1x.com/2023/02/13/pSoKUoD.png)

存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式。存储引擎是基于表而不是基于库的，所以存储引擎也可以被称为表引擎，默认存储引擎为InnoDB

#### InnoDB

支持事务；行级锁，提高并发访问性能；支持外键

xxx.ibd: xxx代表表名，InnoDB 引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm、sdi）、数据和索引

- 参数：innodb_file_per_table，决定多张表共享一个表空间还是每张表对应一个表空间

查看mysql变量：`show variables like 'innodb_file_per_table';`

从idb文件提取表结构数据(在cmd运行)：`ibd2sdi xxx.ibd`

![](https://s1.ax1x.com/2023/02/13/pSoKdFe.png)

#### MyISAM

mysql早期默认存储引擎；不支持事务和外键、支持表锁，不支持行锁、访问速度快

xxx.sdi: 存储表结构信息；xxx.MYD: 存储数据；xxx.MYI: 存储索引

#### Memory

表数据存在内存中，受硬件、断电问题的影响，只能将这些表作为临时表或缓存

存放在内存中，速度快；hash索引；xxx.sdi: 存储表结构信息

#### 存储引擎特点

| 特点         | InnoDB              | MyISAM | Memory |
| ------------ | ------------------- | ------ | ------ |
| 存储限制     | 64TB                | 有     | 有     |
| 事务安全     | 支持                | -      | -      |
| 锁机制       | 行锁                | 表锁   | 表锁   |
| B+tree索引   | 支持                | 支持   | 支持   |
| Hash索引     | -                   | -      | 支持   |
| 全文索引     | 支持（5.6版本之后） | 支持   | -      |
| 空间使用     | 高                  | 低     | N/A    |
| 内存使用     | 高                  | 低     | 中等   |
| 批量插入速度 | 低                  | 高     | 高     |
| 支持外键     | 支持                | -      | -      |

#### 存储引擎的选择

在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎；对于复杂的应用系统，还可以根据实际情况选择多种存储引擎进行组合

- InnoDB：若应用对事物的完整性有较高要求，在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很多更新、删除操作
- MyISAm：若应用以读和插入操作为主，只有很少的更新和删除操作，且对事物完整性、并发性要求不高
- Memory：将所有数据保存在内存中，访问速度快，常用于临时表及缓存，对表的大小有限制，无法保障数据的安全性

电商中足迹和评论适合使用MyISAm，缓存适合用Memory

### 性能分析

#### 查看执行频次

查看当前数据库的 INSERT, UPDATE, DELETE, SELECT 访问频次：

- 查看全局数据：`show global status like 'Com_______';`
- 查看当前会话：`show session status like 'Com_______';`

#### 慢查询日志

记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有SQL语句的日志

```
MySQL的慢查询日志默认没有开启，需要在MySQL的配置文件（/etc/my.cnf）中配置如下信息：
	# 开启慢查询日志开关
	slow_query_log=1
	# 设置慢查询日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
	long_query_time=2
更改后记得重启MySQL服务，日志文件位置：/var/lib/mysql/localhost-slow.log
```

- 查看慢查询日志开关状态：`show variables like 'slow_query_log';`

#### profile

在做SQL优化时了解时间耗费在哪，通过`have_profiling`参数可以看到当前mysql是否支持profile操作：

`select @@have_profiling;`

```mysql
-- profilling默认关闭，可以通过set语句在session/global级别开启profilling
set profiling = 1;
-- 查看所有语句的耗时
show profiles;
-- 查看指定query_id的SQL语句各个阶段的耗时
show profile for query query_id;
-- 查看指定query_id的SQL语句CPU的使用情况
show profile cpu for query query_id;
```

#### explain

EXPLAIN 或者 DESC 命令获取 MySQL 如何执行 SELECT 语句的信息，包括在 SELECT 语句执行过程中表如何连接和连接的顺序

- 获取表是否顺序读取
- 获取数据读取操作的操作类型
- 获取哪些索引可以使用
- 获取哪些索引实际被使用
- 表之间的引用
- 每张表有多少行被优化器查询

`explain select 字段列表 from 表名 where 条件 ` 

| 字段         | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| id           | select查询的序列号，表示查询中执行select子句或操作表的顺序（id相同，执行顺序从上到下；id不同，值越大越先执行） |
| select_type  | 表示select的类型，常见取值有：SIMPLE(简单表，即不适用表连接或子查询)、PRIMARY(主查询，即外层查询)、UNION(UNION中的第二个或后面的查询语句)、SUBQUERY(SELECT/WHERE之后包含了子查询)等 |
| type         | 连接类型，性能由好到差的连接类型为：NULL、system、const、eq_ref、ref、range、index、all |
| possible_key | 可能应用在这张表上的索引，一个或多个                         |
| key          | 实际使用的索引，若为NULL，则没有索引                         |
| key_len      | 索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好 |
| rows         | mysql认为必须要执行的行数，在InnoDB的表中该值可能不准确      |
| filtered     | 表示返回结果的行数占需读取行数的百分比，值越大越好           |

### 索引

帮助 MySQL **高效获取数据**的**数据结构（有序）**。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用数据，这样就可以在这些数据结构上实现高级查询算法

**优点：**

- 提高数据检索效率，降低数据库io成本，类似于书的目录
- 通过索引列对数据排序，降低数据排序成本和CPU的消耗

**缺点：**

- 索引列也要占用空间
- 索引大大提高了查询效率，但降低了更新的速度，如INSERT、UPDATE、DELETE

#### 索引结构

| 索引结构            | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| B+Tree              | 最常见的索引类型，大部分引擎都支持B+树索引                   |
| Hash                | 底层数据结构是用哈希表实现，只有精确匹配索引列的查询才有效，不支持范围查询 |
| R-Tree(空间索引)    | 空间索引是 MyISAM 引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少 |
| Full-Text(全文索引) | 是一种通过建立倒排索引，快速匹配文档的方式，类似于 Lucene, Solr, ES |

| 索引       | InnoDB        | MyISAM | Memory |
| ---------- | ------------- | ------ | ------ |
| B+Tree索引 | 支持          | 支持   | 支持   |
| Hash索引   | 不支持        | 不支持 | 支持   |
| R-Tree索引 | 不支持        | 支持   | 不支持 |
| Full-text  | 5.6版本后支持 | 支持   | 不支持 |

##### B-Tree

![](https://s1.ax1x.com/2023/02/13/pSoMZpd.md.png)

二叉树的缺点可以用红黑树来解决：

![](https://s1.ax1x.com/2023/02/13/pSoMEfH.md.png)

红黑树也存在大数据量情况下，层级较深，检索速度慢的问题。

为了解决上述问题，可以使用 B-Tree 结构。
B-Tree (多路平衡查找树) 以一棵最大度数（max-degree，指一个节点的子节点个数）为5（5阶）的 b-tree 为例（每个节点最多存储4个key，5个指针）

![](https://s1.ax1x.com/2023/02/13/pSoK5Ss.png)

> B-Tree 的数据插入过程动画参照：https://www.bilibili.com/video/BV1Kr4y1i7ru?p=68
> 演示地址：https://www.cs.usfca.edu/~galles/visualization/BTree.html

##### B+Tree

![](https://s1.ax1x.com/2023/02/13/pSoKIln.md.png)

> 演示地址：https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html

与 B-Tree 的区别：

- 所有的数据都会出现在叶子节点
- 叶子节点形成一个单向链表

MySQL 索引数据结构对经典的 B+Tree 进行了优化。在原 B+Tree 的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的 B+Tree，提高区间访问的性能

![](https://s1.ax1x.com/2023/02/13/pSoKoyq.md.png)

##### Hash

哈希索引即采用一定的hash算法，将键值换算成新的hash值，映射到对应的槽位上，然后存储在hash表中；如果两个（或多个）键值映射到一个相同的槽位上就产生了hash冲突（也称为hash碰撞），可以通过链表来解决

![](https://s1.ax1x.com/2023/02/13/pSoKTO0.png)

- Hash索引只能用于对等比较（=、in），不支持范围查询（betwwn、>、<、...）
- 无法利用索引完成排序操作
- 查询效率高，通常只需要一次检索就可以了，效率通常要高于 B+Tree 索引
- 存储引擎支持：Memory、InnoDB（具有自适应hash功能，hash索引是存储引擎根据 B+Tree 索引在指定条件下自动构建的）

#### 索引分类

| 分类     | 含义                                                 | 特点                     | 关键字   |
| -------- | ---------------------------------------------------- | ------------------------ | -------- |
| 主键索引 | 针对于表中主键创建的索引                             | 默认自动创建，只能有一个 | PRIMARY  |
| 唯一索引 | 避免同一个表中某数据列中的值重复                     | 可以有多个               | UNIQUE   |
| 常规索引 | 快速定位特定数据                                     | 可以有多个               |          |
| 全文索引 | 全文索引查找的是文本中的关键词，而不是比较索引中的值 | 可以有多个               | FULLTEXT |

##### MyIsam索引

###### 主键索引

表user的索引存储在索引文件`user.MYI`中，数据文件存储在数据文件 `user.MYD`中

```mysql
select * from user where id = 28;

# 先在主键树中从根节点开始检索，将根节点加载到内存，比较28<75，走左路（1次磁盘IO）
# 将左子树节点加载到内存中，比较16<28<47，向下检索（1次磁盘IO）
# 检索到叶节点，将节点加载到内存中遍历，比较16<28，18<28，28=28。查找到值等于30的索引项（1次磁盘IO）
# 从索引项中获取磁盘地址，然后到数据文件user.MYD中获取对应整行记录（1次磁盘IO）
# 将记录返回给客户端

# 磁盘IO次数：3次索引检索+记录数据检索
```

![](https://s1.ax1x.com/2023/02/13/pSopdPO.md.png)

```mysql
select * from user where id between 28 and 47;

# 先在主键树中从根节点开始检索，将根节点加载到内存，比较28<75，走左路（1次磁盘IO）
# 将左子树节点加载到内存中，比较16<28<47，向下检索（1次磁盘IO）
# 检索到叶节点，将节点加载到内存中遍历比较16<28，18<28，28=28<47。查找到值等于28的索引项；根据磁盘地址从数据文件中获取行记录缓存到结果集中（1次磁盘IO）；查询语句时范围查找，需要向后遍历底层叶子链表，直至到达最后一个不满足筛选条件
# 向后遍历底层叶子链表，将下一个节点加载到内存中，遍历比较，28<47=47，根据磁盘地址从数据文件中获取行记录缓存到结果集中（1次磁盘IO）
# 最后得到两条符合筛选条件，将查询结果集返给客户端

# 磁盘IO次数：4次索引检索+记录数据检索
```

![](https://s1.ax1x.com/2023/02/13/pSoP2Ix.png)

###### 辅助索引

在 MyISAM 中,辅助索引和主键索引的结构是一样的，没有任何区别，叶子节点的数据存储的都是行记录的磁盘地址。只是主键索引的键值是唯一的，而辅助索引的键值可以重复；查询数据时，由于辅助索引的键值不唯一，可能存在多个拥有相同的记录，所以即使是等值查询，也需要按照范围查询的方式在辅助索引树中检索数据

##### InnoDB索引

###### 主键索引（聚簇索引）

每个InnoDB表都有一个聚簇索引 ，聚簇索引使用B+树构建，叶子节点存储的数据是整行记录；当一个表没有创建主键索引时，InnoDB会自动创建一个ROWID字段来构建聚簇索引

> 1、在表上定义主键PRIMARY KEY，InnoDB将主键索引用作聚簇索引
>
> 2、若表没有定义主键，InnoDB会选择第一个不为NULL的唯一索引列用作聚簇索引
>
> 3、若以上两个都没有，InnoDB 会用一个6 字节长整型的隐式字段 ROWID字段构建聚簇索引。该ROWID字段会在插入新行时自动递增

除聚簇索引外的所有索引都称为辅助索引；辅助索引中的叶子节点存储的数据是该行的主键值，检索时，InnoDB使用此主键值在聚簇索引中搜索行记录

```mysql
# 以user_innodb为例，user_innodb的id列为主键，age列为普通索引
CREATE TABLE `user_innodb`
(
  `id`       int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) DEFAULT NULL,
  `age`      int(11)     DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_age` (`age`) USING BTREE
) ENGINE = InnoDB;
```

![](https://s1.ax1x.com/2023/02/13/pSoFq8f.png)

![](https://s1.ax1x.com/2023/02/13/pSoknaR.png)

```mysql
# 等值查询数据
select * from user_innodb where id = 28;

# 先在主键树中从根节点开始检索，将根节点加载到内存，比较28<75，走左路（1次磁盘IO）
# 将左子树节点加载到内存中，比较16<28<47，向下检索（1次磁盘IO）
# 检索到叶节点，将节点加载到内存中遍历，比较16<28，18<28，28=28。查找到值等于28的索引项，直接可以获取整行数据。将改记录返回给客户端（1次磁盘IO）

# 磁盘IO数量：3次
```

![](https://s1.ax1x.com/2023/02/13/pSokuI1.png)

###### 辅助索引

除聚簇索引之外的所有索引都称为辅助索引，InnoDB的辅助索引只会存储主键值而非磁盘地址

![](https://s1.ax1x.com/2023/02/13/pSokmZ9.png)

底层叶子节点的按照（age，id）的顺序排序，先按照age从小到大排序，age列相同时按照id列排序；使用辅助索引需要检索两遍索引：首先检索辅助索引获得主键，然后使用主键到主索引中检索记录（该过程称为回表查询）

```mysql
select * from t_user_innodb where age=19;

# 磁盘IO数：辅助索引3次+获取记录回表3次
```

![](https://s1.ax1x.com/2023/02/13/pSokMPx.png)

##### 联合索引

`频繁使用的列、区分度高的列放在前面，频繁使用代表索引利用率高，区分度高代表筛选粒度大,更好地使用覆盖索引优化`

表 abc_innodb，id为主键索引，创建了一个联合索引idx_abc(a,b,c)；（遵循最左匹配原则；先比较a，b在a等值时有序，c在ab等值时有序）

```mysql
CREATE TABLE `abc_innodb`
(
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `a`  int(11)     DEFAULT NULL,
  `b`  int(11)     DEFAULT NULL,
  `c`  varchar(10) DEFAULT NULL,
  `d`  varchar(10) DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_abc` (`a`, `b`, `c`)
) ENGINE = InnoDB;

select * from abc_innodb order by a, b, c, id;
```

![](https://s1.ax1x.com/2023/02/13/pSoEZNR.png)

![](https://s1.ax1x.com/2023/02/13/pSoEVE9.png)

##### 覆盖索引

常用的优化手段，因为在使用附辅助索引时只可以拿到主键值，还需根据主键进行主键索引获取数据；但若如上所示只需abc字段时，则查询到组合索引的叶子节点就可以返回了，不需要回表查询

#### 语法

- 创建索引：`create [ unique | fulltext ] index index_name on table_name (index_col_name, ...);`
- 查看索引：`show index from table_name;`
- 删除索引：`drop index index_name on table_name;`

```mysql
-- name字段为姓名字段，该字段值可能重复，为该字段创建索引
create index idx_user_name on tb_user(name);
-- phone手机号字段的值非空且唯一，为该字段创建唯一索引
create unique index idx_user_phone on tb_user(phone);
-- 为profession，age，status创建联合索引
create index idx_user_pro_age_stat on tb_user(profession,age,status);
-- 为email建立合适的索引提升查询效率
create index idx_user_email on tb_user(email);
-- 删除索引
drop index idx_user_email on ta_user;
```

#### 索引失效情况

1. 在索引列上进行运算操作或使用了函数，索引将失效，如：`explain select * from tb_user where substring(phone, 10, 2) = '15';`
2. 字段类型不同，如：`explain select * from tb_user where phone = 17799990015;`，phone的值没有加引号
3. 模糊查询中，如果仅仅是尾部模糊匹配，索引不会是失效；如果是头部模糊匹配，索引失效。如：`explain select * from tb_user where profession like '%工程';`，前后都有 % 也会失效
4. 联合索引不满足最左匹配原则

#### SQL提示

其中`use`是建议，实际使用哪个索引MySQL还会自己权衡运行速度去更改，force即无论如何都强制使用该索引

```mysql
-- 使用索引
explain select * from tb_user use index(idx_user_pro) where profession="软件工程";
-- 不使用哪个索引
explain select * from tb_user ignore index(idx_user_pro) where
profession="软件工程";
-- 必须使用哪个索引
explain select * from tb_user force index(idx_user_pro) where
profession="软件工程";
```

#### 设计原则

- 针对数据量大且查询比较频繁的表建立索引
- 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引
- 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高
- 若是字符串类型的字段，字段长度较长，可以针对字段的特点建立前缀索引
- 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率
- 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价就越大，会影响增删改的效率
- 如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。当优化器知道每列是否包含NULL值时，可以更好地确定哪个索引最有效地用于查询

### SQL优化

#### 基础SQL优化

- 尽量使用数值替代字符串类型
- 索引不适合建在有大量重复数据的字段上
- 避免在where子句中使用`!=`或`<>`操作符，可能会让索引失效
- 用`union all`或分成两个子句替换`or`来连接条件
- 使用默认值代替`null`

#### 高级SQL优化

##### 插入数据

- 采用批量插入（一次插入的数据不建议超过1000条）
- 手动提交事务
- 主键顺序插入

若一次性需要插入大批量数据，使用insert语句插入性能较低，此时可以用MySQL数据库提供的load指令插入

```mysql
# 客户端连接服务端时，加上参数 --local-infile（这行在bash/cmd界面输入）
mysql --local-infile -u root -p
# 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set global local_infile = 1；
select @@local_infile;
# 执行load指令将准备好的数据加载到表结构中
load data local infile '/root/sql1.log' into table 'tb_user'
fields terminated by ',' lines terminated by '\n';
```

##### 删除数据

避免同时修改或删除过多数据，cpu利用率过高造成锁表操作，从而影响对数据库的访问

```mysql
# 反例
delete from student where id < 100000;

for (User user:list) {
	delete from student;
}

# 正例
# 分批进行删除，如每次500
for() {
	delete student where id < 500;
}

delete student where id >= 500 and id < 1000;
```

##### 先过滤再使用group by分组

##### 排序字段创建索引（where和order by常出现的字段）

##### 主键优化

- 满足业务需求的情况下，尽量降低主键的长度
- 插入数据时，尽量选择顺序插入，选择使用 AUTO_INCREMENT 自增主键
- 尽量不要使用 UUID 做主键或者是其他的自然主键，如身份证号
- 业务操作时，避免对主键的修改

## 面试题

1. 为什么 InnoDB 存储引擎选择使用 B+Tree 索引结构？

   - 相对于二叉树，层级更少，搜索效率高
   - 对于 B-Tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针也跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低
   - 相对于 Hash 索引，B+Tree 支持范围匹配及排序操作

2. 某表包含字段（id、username、password、status），由于数据量大，需要对以下SQL语句进行优化：

   `select id,username,password from tb_user where username='itcast';`

   `解：`给username和password字段建立联合索引，则不需要回表查询，直接覆盖索引

3. 