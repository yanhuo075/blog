---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL AVG() 函数
order: 5
---

## SQL AVG() 函数

------

### AVG() 函数

AVG() 函数返回数字列的平均值。



### AVG() 语法

```
SELECT AVG(column_name)
FROM table_name
WHERE condition;
```



## 演示数据库

------

在本教程中，我们将使用著名的 Northwind 样本数据库。

下面是选自 "Products" 表的数据：

| ProductID | ProductName                  | SupplierID | CategoryID | Unit                | Price |
| :-------- | :--------------------------- | :--------- | :--------- | :------------------ | :---- |
| 1         | Chais                        | 1          | 1          | 10 boxes x 20 bags  | 18    |
| 2         | Chang                        | 1          | 1          | 24 - 12 oz bottles  | 19    |
| 3         | Aniseed Syrup                | 1          | 2          | 12 - 550 ml bottles | 10    |
| 4         | Chef Anton's Cajun Seasoning | 2          | 2          | 48 - 6 oz jars      | 22    |
| 5         | Chef Anton's Gumbo Mix       | 2          | 2          | 36 boxes            | 21.35 |



## SQL AVG() 实例

------

以下SQL语句查找所有产品的平均价格：

示例

```
SELECT AVG(Price)
FROM Products;
```

下面的 SQL 语句选择价格高于平均价格的 "ProductName" 和 "Price" 记录：

示例

```
SELECT ProductName, Price FROM Products
WHERE Price>(SELECT AVG(Price) FROM Products);
```

