---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL 空值
order: 10
---

## 什么是SQL NULL值？

------

SQL 中，**NULL** 用于表示缺失的值。数据表中的 NULL 值表示该值所处的字段为空。

具有NULL值的字段是没有值的字段。

如果表中的字段是可选的，则可以插入新记录或更新记录而不向该字段添加值。然后，该字段将被保存为NULL值。

值为 NULL 的字段没有值。尤其要明白的是，NULL 值与 0 或者包含空白（spaces）的字段是不同的。

> **注意**：理解NULL值与零值或包含空格的字段不同是非常重要的。具有NULL值的字段是在记录创建期间留空的字段！

## 如何测试NULL值？

------

使用比较运算符（例如=，<或<>）来测试NULL值是不可行的。

我们将不得不使用IS NULL和IS NOT NULL运算符。

### IS NULL语法

```
SELECT column_names
FROM table_name
WHERE column_name IS NULL;
```

### IS NOT NULL语法

```
SELECT column_names
FROM table_name
WHERE column_name IS NOT NULL;
```

## 演示数据库

------

假设我们有以下的"人员"表：

| ID   | LastName | FirstName | Address            | City     |
| :--- | :------- | :-------- | :----------------- | :------- |
| 1    | Doe      | John      | 542 W. 27th Street | New York |
| 2    | Bloggs   | Joe       |                    | London   |
| 3    | Roe      | Jane      |                    | New York |
| 4    | Smith    | John      | 110 Bishopsgate    | London   |

假设"人员"表中的"Address"列是可选的。如果插入的记录没有"Address"值，则"Address"列将以空值保存。

## IS NULL运算符

------

以下SQL语句使用IS NULL运算符来列出所有没有地址的人员：

```
SELECT LastName, FirstName, Address FROM Persons
WHERE Address IS NULL;
```

结果集将如下所示：

| LastName | FirstName | Address |
| :------- | :-------- | :------ |
| Bloggs   | Joe       |         |
| Roe      | Jane      |         |

> **提示**：始终使用IS NULL来查找空值。

## IS NOT NULL运算符

------

以下SQL语句使用IS NOT NULL运算符来列出所有具有地址的人员：

```
SELECT LastName, FirstName, Address FROM Persons
WHERE Address IS NOT NULL;
```

结果集将如下所示：

| LastName | FirstName | Address            |
| :------- | :-------- | :----------------- |
| Doe      | John      | 542 W. 27th Street |
| Smith    | John      | 110 Bishopsgate    |

## 语法：

------

创建表的时候，NULL 的基本语法如下：

```
SQL> CREATE TABLE CUSTOMERS(
   ID   INT              NOT NULL,
   NAME VARCHAR (20)     NOT NULL,
   AGE  INT              NOT NULL,
   ADDRESS  CHAR (25) ,
   SALARY   DECIMAL (18, 2),       
   PRIMARY KEY (ID)
);
```

这里，**NOT NULL**表示对于给定列，必须按照其数据类型明确赋值。有两列并没有使用 NOT NULL 来限定，也就是说这些列可以为 NULL。

值为 NULL 的字段是在记录创建的过程中留空的字段。

## 示例：

------

NULL 值会给选取数据带来麻烦。不过，因为 NULL 和其他任何值作比较，其结果总是未知的，所以含有 NULL 的记录不会包含在最终结果里面。

必须使用 **IS NULL** 或者 **IS NOT NULL** 来检测某个字段是否为 NULL。

考虑下面的 CUSTOMERS 数据表，里面包含的记录如下所示：

```
+----+----------+-----+-----------+----------+
| ID | NAME     | AGE | ADDRESS   | SALARY   |
+----+----------+-----+-----------+----------+
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 |
|  2 | Khilan   |  25 | Delhi     |  1500.00 |
|  3 | kaushik  |  23 | Kota      |  2000.00 |
|  4 | Chaitali |  25 | Mumbai    |  6500.00 |
|  5 | Hardik   |  27 | Bhopal    |  8500.00 |
|  6 | Komal    |  22 | MP        |          |
|  7 | Muffy    |  24 | Indore    |          |
+----+----------+-----+-----------+----------+
```

下面是 **IS NOT NULL** 运算符的用法：

```
SQL> SELECT  ID, NAME, AGE, ADDRESS, SALARY
     FROM CUSTOMERS
     WHERE SALARY IS NOT NULL;
```

上面语句的运行结果如下：

```
+----+----------+-----+-----------+----------+
| ID | NAME     | AGE | ADDRESS   | SALARY   |
+----+----------+-----+-----------+----------+
|  1 | Ramesh   |  32 | Ahmedabad |  2000.00 |
|  2 | Khilan   |  25 | Delhi     |  1500.00 |
|  3 | kaushik  |  23 | Kota      |  2000.00 |
|  4 | Chaitali |  25 | Mumbai    |  6500.00 |
|  5 | Hardik   |  27 | Bhopal    |  8500.00 |
+----+----------+-----+-----------+----------+
```

下面是 **IS NULL** 运算符的用法：

```
SQL> SELECT  ID, NAME, AGE, ADDRESS, SALARY
     FROM CUSTOMERS
     WHERE SALARY IS NULL;
```

其运行结果如下：

```
+----+----------+-----+-----------+----------+
| ID | NAME     | AGE | ADDRESS   | SALARY   |
+----+----------+-----+-----------+----------+
|  6 | Komal    |  22 | MP        |          |
|  7 | Muffy    |  24 | Indore    |          |
+----+----------+-----+-----------+----------+
```
