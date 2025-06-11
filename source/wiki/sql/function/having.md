---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL HAVING函数
order: 12
---

## SQL HAVING 子句

------

在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与 Aggregate 函数一起使用。

HAVING子句已添加到SQL中，因为WHERE关键字不能用于聚合函数。

### SQL HAVING 语法

```
SELECT column_name(s)
FROM table_name
WHERE condition
GROUP BY column_name(s)
HAVING condition
ORDER BY column_name(s);
```

## 演示数据库

------

以下是罗斯文示例数据库中"Customers"表的选择：

| CustomerID | CustomerName                       | ContactName        | Address                       | City        | PostalCode | Country |
| :--------- | :--------------------------------- | :----------------- | :---------------------------- | :---------- | :--------- | :------ |
| 1          | Alfreds Futterkiste                | Maria Anders       | Obere Str. 57                 | Berlin      | 12209      | Germany |
| 2          | Ana Trujillo Emparedados y helados | Ana Trujillo       | Avda. de la Constitución 2222 | México D.F. | 05021      | Mexico  |
| 3          | Antonio Moreno Taquería            | Antonio Moreno     | Mataderos 2312                | México D.F. | 05023      | Mexico  |
| 4          | Around the Horn                    | Thomas Hardy       | 120 Hanover Sq.               | London      | WA1 1DP    | UK      |
| 5          | Berglunds snabbköp                 | Christina Berglund | Berguvsvägen 8                | Luleå       | S-958 22   | Sweden  |

## SQL HAVING示例

------

以下SQL语句列出了每个国家/地区的客户数量。只包括超过5位客户的国家/地区：

```
SELECT COUNT(CustomerID), Country
FROM Customers
GROUP BY Country
HAVING COUNT(CustomerID) > 5;
```

以下SQL语句列出每个国家的客户数量，从高到低排序（仅包括拥有超过5名客户的国家/地区）：

```
SELECT COUNT(CustomerID), Country
FROM Customers
GROUP BY Country
HAVING COUNT(CustomerID) > 5
ORDER BY COUNT(CustomerID) DESC;
```

## 演示数据库

------

以下是罗斯文示例数据库中"Orders"表的一个选择：

| OrderID | CustomerID | EmployeeID | OrderDate  | ShipperID |
| :------ | :--------- | :--------- | :--------- | :-------- |
| 10248   | 90         | 5          | 1996-07-04 | 3         |
| 10249   | 81         | 6          | 1996-07-05 | 1         |
| 10250   | 34         | 4          | 1996-07-08 | 2         |

并从"Employees"表中选择：

| EmployeeID | LastName  | FirstName | BirthDate  | Photo      | Notes                       |
| :--------- | :-------- | :-------- | :--------- | :--------- | :-------------------------- |
| 1          | Davolio   | Nancy     | 1968-12-08 | EmpID1.pic | Education includes a BA.... |
| 2          | Fuller    | Andrew    | 1952-02-19 | EmpID2.pic | Andrew received his BTS.... |
| 3          | Leverling | Janet     | 1963-08-30 | EmpID3.pic | Janet has a BS degree....   |

## 更多HAVING示例

------

以下SQL语句列出已注册超过10个订单的员工：

```
SELECT Employees.LastName, COUNT(Orders.OrderID) AS NumberOfOrders
FROM Orders
INNER JOIN Employees ON Orders.EmployeeID = Employees.EmployeeID
GROUP BY LastName
HAVING COUNT(Orders.OrderID) > 10;
```

以下SQL语句列出员工"Davolio"或"Fuller"是否已注册超过25个订单：

```
SELECT Employees.LastName, COUNT(Orders.OrderID) AS NumberOfOrders
FROM Orders
INNER JOIN Employees ON Orders.EmployeeID = Employees.EmployeeID
WHERE LastName = 'Davolio' OR LastName = 'Fuller'
GROUP BY LastName
HAVING COUNT(Orders.OrderID) > 25;
```
