---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL MIN() 函数
order: 3
---

## SQL MIN() 函数

------

### MIN() 函数

MIN() 函数返回所选列的最小值。

### SQL MIN() 语法

```
SELECT MIN(column_name)
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

## SQL MIN() 实例

------

以下SQL语句查找最便宜的产品的价格：

示例

```
SELECT MIN(Price) AS SmallestPrice
FROM Products;
```

