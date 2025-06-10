---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL 语法
order: 3
---

## SQL 语法规则

------

- SQL语句总是以关键字开始，如SELECT、INSERT、UPDATE、DELETE、DROP、CREATE。
- SQL语句以分号结尾。
- SQL不区分大小写，意味着update与UPDATE相同。

## 数据库表

------

数据库通常包含一个或多个表。每个表都用一个名称标识（例如，"Customers"或"Orders"）。该表包含带有数据（行）的记录。
　在本教程中，我们将使用著名的Northwind示例数据库（包括MSAccess和MSSQLServer）。

下面是选自 "Customers" 表的数据：

| CustomerID | CustomerName                       | ContactName        | Address                       | City        | PostalCode | Country |
| :--------- | :--------------------------------- | :----------------- | :---------------------------- | :---------- | :--------- | :------ |
| 1          | Alfreds Futterkiste                | Maria Anders       | Obere Str. 57                 | Berlin      | 12209      | Germany |
| 2          | Ana Trujillo Emparedados y helados | Ana Trujillo       | Avda. de la Constitución 2222 | México D.F. | 05021      | Mexico  |
| 3          | Antonio Moreno Taquería            | Antonio Moreno     | Mataderos 2312                | México D.F. | 05023      | Mexico  |
| 4          | Around the Horn                    | Thomas Hardy       | 120 Hanover Sq.               | London      | WA1 1DP    | UK      |
| 5          | Berglunds snabbköp                 | Christina Berglund | Berguvsvägen 8                | Luleå       | S-958 22   | Sweden  |

上面的表包含五条记录（每一条对应一个客户）和七个列（CustomerID、CustomerName、ContactName、Address、City、PostalCode 和 Country）。

## SQL 语句

------

您需要在数据库上执行的大部分操作都是使用SQL语句完成的。

以下SQL语句选择"Customers"表中的所有记录：

## 实例

SELECT * FROM Customers;

在本教程中，我们将向您解释各种不同的SQL语句。

## 请记住...

------

- SQL 对大小写不敏感：SELECT 与 select 是相同的。
- 在本教程中，我们将以大写形式编写所有SQL关键字。

## SQL 语句后面的分号？

------

- 一些数据库系统需要在每个SQL语句的末尾使用分号。
- 分号是分离数据库系统中每个SQL语句的标准方法，这样您就可以在对服务器的同一请求中执行多个SQL语句。
- 在本教程中，我们将在每个SQL语句的末尾使用分号。

## 一些最重要的 SQL 命令

------

- **SELECT** - 从数据库中提取数据
- **UPDATE** - 更新数据库中的数据
- **DELETE** - 从数据库中删除数据
- **INSERT INTO** - 向数据库中插入新数据
- **CREATE DATABASE** - 创建新数据库
- **ALTER DATABASE** - 修改数据库
- **CREATE TABLE** - 创建新表
- **ALTER TABLE** - 变更（改变）数据库表
- **DROP TABLE** - 删除表
- **CREATE INDEX** - 创建索引（搜索键）
- **DROP INDEX** - 删除索引

### **SELECT语句**

句法：

```
SELECT column_name(s) FROM table_name
```

### SELECT语句和WHERE子句

句法：

```
SELECT [*] FROM [TableName] WHERE [condition1]
```

### SELECT语句与WHERE和/或子句[

句法：

```
SELECT [*] FROM [TableName] WHERE [condition1] [AND [OR]] [condition2]...
```

### SELECT语句与ORDER BY

句法：

```
SELECT column_name()
FROM table_name
ORDER BY column_name() ASC or DESC
```

### SELECT DISTINCT(区分)子句

句法：

```
SELECT DISTINCT column1, column2....columnN
FROM   table_name;
```

### SELECT IN子句

句法：

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name IN (val-1, val-2,...val-N);
```

### SELECT LIKE (类)子句

句法：

```
SELECT column1, column2....columnN
FROM   table_name
WHERE  column_name LIKE { PATTERN };
```

### SELECT COUNT(计数)子句

句法：

```
SELECT COUNT(column_name)
FROM   table_name
WHERE  CONDITION;
```

### SELECT与HAVING子句

句法：

```
SELECT SUM(column_name)
FROM   table_name
WHERE  CONDITION
GROUP BY column_name
HAVING (arithematic function condition);
```

### INSERT INTO语句

句法：

```
INSERT INTO table_name (column, column1, column2, column3, ...)
VALUES (value, value1, value2, value3 ...)
```

### UPDATE语句

句法：

```
UPDATE table_name
SET column=value, column1=value1,...
WHERE someColumn=someValue
```

### DELETE语句

句法：

```
DELETE FROM tableName
WHERE someColumn = someValue
```

### CREATE 语句

句法：

```
CREATE TABLE table_name(
column1 datatype,
column2 datatype,
column3 datatype,
.....
columnN datatype,
PRIMARY KEY( one or more columns )
);
```

### DROP 语句

句法：

```
DROP TABLE table_name;
```

### CREATE INDEX语句

句法：

```
CREATE UNIQUE INDEX index_name
ON table_name ( column1, column2,...columnN);
```

### DROP INDEX语句

句法：

```
ALTER TABLE table_name
DROP INDEX index_name;
```

### DESC语句

句法：

```
DESC table_name;
```

### TRUNCATE 截断表语句

句法：

```
TRUNCATE TABLE table_name;
```

### ALTER TABLE语句

句法：

sql

```
ALTER TABLE table_name {ADD|DROP|MODIFY} column_name {data_ype};
```

### ALTER TABLE语句(对表名重命名)

句法：

```
ALTER TABLE table_name RENAME TO new_table_name;
```

### Use语句

句法：

```
USE database_name;
```

### COMMIT语句

句法：

```
COMMIT;
```

### ROLLBACK语句

句法：

```
ROLLBACK;
```
