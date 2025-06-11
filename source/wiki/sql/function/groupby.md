---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL GROUP BY()函数
order: 11
---

## SQL GROUP BY 语句

------

Aggregate 函数常常需要添加 GROUP BY 语句。

GROUP BY语句通常与集合函数（COUNT，MAX，MIN，SUM，AVG）一起使用，以按一个或多个列对结果集进行分组。

## GROUP BY 语句

------

GROUP BY 语句用于结合 Aggregate 函数，根据一个或多个列对结果集进行分组。

### SQL GROUP BY 语法

```
SELECT column_name(s)
FROM table_name
WHERE condition
GROUP BY column_name(s)
ORDER BY column_name(s);
```

## 演示数据库

------

在本教程中，我们将使用众所周知的 Northwind 样本数据库。

下面是选自 "Customers"表的数据：

| CustomerID | CustomerName                       | ContactName        | Address                       | City        | PostalCode | Country |
| :--------- | :--------------------------------- | :----------------- | :---------------------------- | :---------- | :--------- | :------ |
| 1          | Alfreds Futterkiste                | Maria Anders       | Obere Str. 57                 | Berlin      | 12209      | Germany |
| 2          | Ana Trujillo Emparedados y helados | Ana Trujillo       | Avda. de la Constitución 2222 | México D.F. | 05021      | Mexico  |
| 3          | Antonio Moreno Taquería            | Antonio Moreno     | Mataderos 2312                | México D.F. | 05023      | Mexico  |
| 4          | Around the Horn                    | Thomas Hardy       | 120 Hanover Sq.               | London      | WA1 1DP    | UK      |
| 5          | Berglunds snabbköp                 | Christina Berglund | Berguvsvägen 8                | Luleå       | S-958 22   | Sweden  |

## SQL GROUP BY示例

------

以下SQL语句列出了每个国家/地区的客户数量：

```
SELECT COUNT(CustomerID), Country
FROM Customers
GROUP BY Country;
```

以下SQL语句列出每个国家的客户数量，从高到低排序：

```
SELECT COUNT(CustomerID), Country
FROM Customers
GROUP BY Country
ORDER BY COUNT(CustomerID) DESC;
```

## 演示数据库

------

以下是罗斯文示例数据库中"订单"表的一个选择：

| OrderID | CustomerID | EmployeeID | OrderDate  | ShipperID |
| :------ | :--------- | :--------- | :--------- | :-------- |
| 10248   | 90         | 5          | 1996-07-04 | 3         |
| 10249   | 81         | 6          | 1996-07-05 | 1         |
| 10250   | 34         | 4          | 1996-07-08 | 2         |

并从"Shippers"表中选择：

| ShipperID | ShipperName      |
| :-------- | :--------------- |
| 1         | Speedy Express   |
| 2         | United Package   |
| 3         | Federal Shipping |

## GROUP BY使用JOIN示例

------

以下SQL语句列出了每个发货人发送的订单数量：

```
SELECT Shippers.ShipperName, COUNT(Orders.OrderID) AS NumberOfOrders FROM Orders
LEFT JOIN Shippers ON Orders.ShipperID = Shippers.ShipperID
GROUP BY ShipperName;
```
