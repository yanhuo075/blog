---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL 在表中插入
order: 9
---

## SQL INSERT INTO 语句

------

INSERT INTO 语句用于向表中插入新的数据行。

### SQL INSERT INTO 语法

INSERT INTO 语句可以用两种形式编写。
　第一个表单没有指定要插入数据的列的名称，只提供要插入的值，即可添加一行新的数据：

sql

```
INSERT INTO table_name (column1, column2, column3, ...)
VALUES (value1, value2, value3, ...);
```

第二种，如果要为表中的所有列添加值，则不需要在SQL查询中指定列名称。但是，请确保值的顺序与表中的列顺序相同。INSERT INTO语法如下所示：

sql

```
INSERT INTO table_name
VALUES (value1, value2, value3, ...);
```

## 演示数据库

------

在本教程中，我们将使用著名的 Northwind 示例数据库。

以下是"Customers" 表中的数据：

| CustomerID | CustomerName           | ContactName      | Address                     | City     | PostalCode | Country |
| :--------- | :--------------------- | :--------------- | :-------------------------- | :------- | :--------- | :------ |
| 87         | Wartian Herkku         | Pirkko Koskitalo | Torikatu 38                 | Oulu     | 90110      | Finland |
| 88         | Wellington Importadora | Paula Parente    | Rua do Mercado, 12          | Resende  | 08737-363  | Brazil  |
| 89         | White Clover Markets   | Karl Jablonski   | 305 - 14th Ave. S. Suite 3B | Seattle  | 98128      | USA     |
| 90         | Wilman Kala            | Matti Karttunen  | Keskuskatu 45               | Helsinki | 21240      | Finland |
| 91         | Wolski                 | Zbyszek          | ul. Filtrowa 68             | Walla    | 01-012     | Poland  |

## INSERT INTO 实例

------

假设我们想在"Customers"表中插入一个新行。

我们可以使用以下SQL语句：

## 实例

INSERT INTO Customers (CustomerName, ContactName, Address, City, PostalCode, Country)
VALUES ('Cardinal','Tom B. Erichsen','Skagen 21','Stavanger','4006','Norway');

现在，选自 "Customers" 表的数据如下所示：

| CustomerID | CustomerName           | ContactName      | Address                     | City      | PostalCode | Country |
| :--------- | :--------------------- | :--------------- | :-------------------------- | :-------- | :--------- | :------ |
| 87         | Wartian Herkku         | Pirkko Koskitalo | Torikatu 38                 | Oulu      | 90110      | Finland |
| 88         | Wellington Importadora | Paula Parente    | Rua do Mercado, 12          | Resende   | 08737-363  | Brazil  |
| 89         | White Clover Markets   | Karl Jablonski   | 305 - 14th Ave. S. Suite 3B | Seattle   | 98128      | USA     |
| 90         | Wilman Kala            | Matti Karttunen  | Keskuskatu 45               | Helsinki  | 21240      | Finland |
| 91         | Wolski                 | Zbyszek          | ul. Filtrowa 68             | Walla     | 01-012     | Poland  |
| 92         | Cardinal               | Tom B. Erichsen  | Skagen 21                   | Stavanger | 4006       | Norway  |

| **注意到了吗？** **我们没有将任何号码插入 CustomerID 字段。** CustomerID列是一个字段，在将新记录插入到表中时自动生成。 |
| ------------------------------------------------------------ |

## 仅在指定的列中插入数据

------

我们还可以只在指定的列中插入数据。

以下SQL语句插入一个新行，但只在"CustomerName"、"City"和"Country"列中插入数据（CustomerID字段将自动更新）：

示例：

```
INSERT INTO Customers (CustomerName, City, Country)
VALUES ('Cardinal', 'Stavanger', 'Norway');
```

现在，选自 "Customers" 表的数据如下所示：

| CustomerID | CustomerName           | ContactName      | Address                     | City      | PostalCode | Country |
| :--------- | :--------------------- | :--------------- | :-------------------------- | :-------- | :--------- | :------ |
| 87         | Wartian Herkku         | Pirkko Koskitalo | Torikatu 38                 | Oulu      | 90110      | Finland |
| 88         | Wellington Importadora | Paula Parente    | Rua do Mercado, 12          | Resende   | 08737-363  | Brazil  |
| 89         | White Clover Markets   | Karl Jablonski   | 305 - 14th Ave. S. Suite 3B | Seattle   | 98128      | USA     |
| 90         | Wilman Kala            | Matti Karttunen  | Keskuskatu 45               | Helsinki  | 21240      | Finland |
| 91         | Wolski                 | Zbyszek          | ul. Filtrowa 68             | Walla     | 01-012     | Poland  |
| 92         | Cardinal               | null             | null                        | Stavanger | null       | Norway  |

## 使用另一个表填充一个表

------

您可以通过另一个表上的SELECT语句查询出来的字段值，然后将数据填充到本表中，条件是另一个表所查询的字段与本表要插入数据的字段是一一对应的。

```
INSERT INTO first_table_name [(column1, column2, ... columnN)] 
SELECT column1, column2, ...columnN 
FROM second_table_name
[WHERE condition];
```
