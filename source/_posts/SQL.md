---
title: Basic of SQL
date: 2019-07-19 19:21:02
tags: [MySQL,SQL]
categories: [SQL]
copyright: true
description: <center>假期项目MySQL学习总结</center>
photo: "https://user-images.githubusercontent.com/32346935/61532017-28840f80-aa5b-11e9-95d1-25ffc78f82a3.jpg"
---
---

# SQL基础总结

## SQL语言分类

1、DML（Data Manipulation Language）:数据操纵语句，用于添 加、删除、修改、查询数据库记录，并检查数据完整性。

INSERT：添加数据到数据库中
UPDATE：修改数据库中的数据
DELETE：删除数据库中的数据
SELECT：选择（查询）数据


2、DDL（Data Definition Language）:数据定义语句，用于库和 表的创建、修改、删除。

CREATE TABLE：创建数据库表
ALTER TABLE：更改表结构、添加、删除、修改列长度
DROP TABLE：删除表

3、DCL（Data Control Language）:数据控制语句，用于定义用 户的访问权限和安全级别。

COMMIT：提交事务处理
ROLLBACK：事务处理回退
SAVEPOINT：设置保存点
LOCK：对数据库的特定部分进行锁定

## 数据查询

### SELECT

```sql
SELECT *|{[DISTINCT] column|expression [alias],...}
FROM table;
```

SELECT：标识选择哪些列
FROM：标识从哪个表中选择

**列的别名**有三种方式，例子如下：

```sql
SELECT last_name AS name, commission_pct comm
FROM employees;

SELECT last_name "Name", salary*12  
FROM employees;

SELECT commission_pct comm
FROM employees;
```

### DESCRIBE

查询表结构：

```sql
DESC[RIBE] tablename;
```

### WHERE

`where`子句用于对查询的数据进行过滤：

```sql
SELECT    *|{[DISTINCT] column|expression [alias],...}
FROM    table  
[WHERE    condition(s)];
```

WHERE子句紧随FROM子句，除了可以用`>\<\!=`等常用的比较运算符外，还可以使用下列操作符进行数据过滤：

操作符|含义
-----|----
BETWEEN...AND...|在两个值之间（包含边界）
IN(set)|等于值列表中的一个
LIKE|模糊查询
IS NULL|空值

**PS**:WHERE子句中不能使用别名，不能使用组函数

### ORDER BY

子句`order by`用于排序
asc:升序
desc:降序

```sql
SELECT XXX FROM XXX
ORDER BY XXX 

SELECT XXX FROM XXX
ORDER BY XXX DESC

/*多个列排序，分主次，在前的为主*/
SELECT XXX FROM XXX
ORDER BY XXX DESC，XXX
```

### 组函数

组函数作用于一组数据，并对一组数据返回一个值。其类型有：

```
AVG()
COUNT()
MAX()
MIN()
SUM()
```

### GROUP BY

GROUP BY子句将表中的数据分成若干组

```sql
SELECT column, group_function(column)  
FROM table
[WHERE condition]
[GROUP BY group_by_expression]  
[ORDER BY column];
```

### HAVING

```sql
SELECT column, group_function(column)  
FROM table
[WHERE condition]
[GROUP BY group_by_expression] 
[HAVING group_condition] 
[ORDER BY column];
```

使用HAVING子句的几个条件：
1. 行已经被分组
2. 使用了组函数
3. 满足HAVING子句中条件的分组将被显示

### JOIN ON

多表连接，其可以有几种分类：
内连接：`[INNER] join on`
外连接：
·`左外连接`：left [outer] join on
·`右外连接`：right [outer] join on

## 常见函数

### 字符函数

```
LOWER  UPPER

CONCAT  SUBSTR  LENGTH  INSTR
LPAD | RPAD  TRIM  REPLACE
```

### 数字函数

```
ROUND: 四舍五入
TRUNCATE:截断
MOD: 求余
```

### 日期函数

```
now()：获取当前日期
str_to_date: 将日期格式的字符转换成指定格式的日期
date_format:将日期转换成字符
```

序号|格式符|功能
---|------|----
1|%Y|四位的年份
2|%y|2位的年份
3|%m|月份(01,02…11,12)
4|%c|月份（1,2,…11,12）
5|%d|日（01,02,…）
6|%H|小时（24小时制）
7|%h|小时（12小时制）
8|%i|分钟（00,01…59）
9|%s|秒（00,01,…59）

### CASE表达式

```sql
CASE expr WHEN comparison_expr1 THEN return_expr1  [WHEN comparison_expr2 THEN return_expr2  WHEN comparison_exprn THEN return_exprn
ELSE else_expr]
END
```

## 子查询

子查询是出现在其他语句内部的select语句。
单行子查询使用单行比较操作符；
多行子查询使用多行比较操作符；

### 多行子查询的比较操作符
操作符|含义
-----|----
IN/NOT IN|等于列表中的任意一个
ANY SOME|和子查询返回的某一个值比较
ALL|和子查询返回的所有值比较

## 创建和管理表

### CREATE TABLE

```sql
CREATE TABLE [schema.]table
(column datatype [DEFAULT expr][, ...]);
```

还可以使用子查询来创建表、复制现有表

### ALTER TABLE

使用 ALTER TABLE 语句可以实现：
·向已有的表中添加列（ADD）
·修改现有表中的列（MODIFY）
·删除现有表中的列（DROP）
·重命名现有表中的列（CHANGE）

### 删除表

DROP:删除数据和结够，不能回滚
TRUNCATE TABLE:清空表中所有的数据
DELETE:删除数据，可以回滚，在commit后才真正删除

### RENAME

```sql
ALTER table dept  
RENAME TO detail_dept;  
```

## 数据类型

### 数值类型

分为整型、小数、位类型

### 字符类型

分为char和varchar、binary和varbinary、enum和set类型

### 日期类型

略

## 约束和分页

### 约束

六种约束：

```
NOT NULL 非空约束，规定某个字段不能为空
UNIQUE	唯一约束，规定某个字段在整个表中是唯一的
PRIMARY KEY 主键(非空且唯一)
FOREIGN KEY 外键
CHECK	检查约束
DEFAULT	默认值
```

### 分页

```sql
/*9：从第几条数据的下一条数据开始取，10：取几条数据*/
select * from employees limit 9, 10;
select * from employees limit 10, offset 9;

/*pageSize:每页显示多少条数据，pageNumber:页码*/
/*返回第pageNumber页 每条页数，为pageSize 的sql语句为：
select * from xxx limit (pageNmuber - 1)*pageSize, pageSize;*/

/*返回第5页，每页数据为20条的数据*/
select * from employees limit 80, 20;
```

## 事务

事务：事务由单独单元的一个或多个SQL语句组成，在这 个单元中，每个MySQL语句是相互依赖的。而整个单独单 元作为一个不可分割的整体，如果单元中某条SQL语句一 旦执行失败或产生错误，整个单元将会回滚。所有受到影 响的数据将返回到事物开始以前的状态；如果单元中的所 有SQL语句均执行成功，则事物被顺利执行。

