---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL LEN()函数
order: 19
---

## SQL LEN() 函数

------

LEN() 函数返回文本字段中值的长度。

### SQL LEN() 语法

```
SELECT LEN(column_name) FROM table_name;
```

## 演示数据库

------

在本教程中，我们将使用众所周知的 Northwind 样本数据库。

下面是选自 "Customers" 表的数据：

| CustomerID | CustomerName                       | ContactName        | Address                       | City        | PostalCode | Country |
| :--------- | :--------------------------------- | :----------------- | :---------------------------- | :---------- | :--------- | :------ |
| 1          | Alfreds Futterkiste                | Maria Anders       | Obere Str. 57                 | Berlin      | 12209      | Germany |
| 2          | Ana Trujillo Emparedados y helados | Ana Trujillo       | Avda. de la Constitución 2222 | México D.F. | 05021      | Mexico  |
| 3          | Antonio Moreno Taquería            | Antonio Moreno     | Mataderos 2312                | México D.F. | 05023      | Mexico  |
| 4          | Around the Horn                    | Thomas Hardy       | 120 Hanover Sq.               | London      | WA1 1DP    | UK      |
| 5          | Berglunds snabbköp                 | Christina Berglund | Berguvsvägen 8                | Luleå       | S-958 22   | Sweden  |

## SQL LEN() 实例

------

下面的 SQL 语句从 "Customers" 表中选取 "CustomerName" 和 "Address" 列中值的长度：

示例

```
SELECT CustomerName,LEN(Address) as LengthOfAddress
FROM Customers;
```

