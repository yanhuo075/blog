---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL Date函数
order: 38
---

## SQL Date 函数

------

> **注意：**当我们处理日期时，最困难的任务可能是确保插入日期的格式与数据库中日期列中的格式相匹配。

只要您的数据仅包含日期的一部分，运行查询就不会成为问题。然而，当涉及到时间时，情况会稍微复杂一些。

在讨论日期查询的复杂性之前，让我们看看最重要的内置日期处理程序。

## MySQL Date 函数

------

下表列出了 MySQL 中最重要的内置日期函数：

| 函数 | 描述                                |
| :--- | :---------------------------------- |
|      | 返回当前的日期和时间                |
|      | 返回当前的日期                      |
|      | 返回当前的时间                      |
|      | 提取日期或日期/时间表达式的日期部分 |
|      | 返回日期/时间的单独部分             |
|      | 向日期添加指定的时间间隔            |
|      | 从日期减去指定的时间间隔            |
|      | 返回两个日期之间的天数              |
|      | 用不同的格式显示日期/时间           |

## SQL Server Date 函数

------

下表列出了SQL 服务器中最重要的内置日期函数：

| 函数 | 描述                             |
| :--- | :------------------------------- |
|      | 返回当前的日期和时间             |
|      | 返回日期/时间的单独部分          |
|      | 在日期中添加或减去指定的时间间隔 |
|      | 返回两个日期之间的时间           |
|      | 用不同的格式显示日期/时间        |

## SQL Date 数据类型

------

**　MySQL** 使用下列数据类型在数据库中存储日期或时间值：

- DATE - 格式：YYYY-MM-DD
- DATETIME - 格式：YYYY-MM-DD HH:MM:SS
- TIMESTAMP - 格式：YYYY-MM-DD HH:MM:SS
- YEAR - 格式：YYYY 或 YY

**　SQL Server** 使用下列数据类型在数据库中存储日期或时间值：

- DATE - 格式：YYYY-MM-DD
- DATETIME - 格式：YYYY-MM-DD HH:MM:SS
- SMALLDATETIME - 格式：YYYY-MM-DD HH:MM:SS
- TIMESTAMP - 格式：唯一的数字

**　注释：**在数据库中创建新表时，需要为该列选择数据类型！

如需了解所有可用的数据类型，请访问我们完整的 。

## SQL 日期处理

------

**注意：**如果您不涉及时间部分，那么我们可以轻松比较两个日期！

假设我们有以下"订单"表：

| OrderId | ProductName            | OrderDate  |
| :------ | :--------------------- | :--------- |
| 1       | Geitost                | 2008-11-11 |
| 2       | Camembert Pierrot      | 2008-11-09 |
| 3       | Mozzarella di Giovanni | 2008-11-11 |
| 4       | Mascarpone Fabioli     | 2008-10-29 |

现在，我们希望从上表中选取 OrderDate 为 "2008-11-11" 的记录。

我们使用下面的 SELECT 语句：

```
SELECT * FROM Orders WHERE OrderDate='2008-11-11'
```

结果集如下所示：

| OrderId | ProductName            | OrderDate  |
| :------ | :--------------------- | :--------- |
| 1       | Geitost                | 2008-11-11 |
| 3       | Mozzarella di Giovanni | 2008-11-11 |

现在，假设 "Orders" 表如下所示（请注意 "OrderDate" 列中的时间部分）：

| OrderId | ProductName            | OrderDate           |
| :------ | :--------------------- | :------------------ |
| 1       | Geitost                | 2008-11-11 13:23:44 |
| 2       | Camembert Pierrot      | 2008-11-09 15:45:21 |
| 3       | Mozzarella di Giovanni | 2008-11-11 11:12:01 |
| 4       | Mascarpone Fabioli     | 2008-10-29 14:56:59 |

如果我们使用和上面一样的 SELECT 语句：

```
SELECT * FROM Orders WHERE OrderDate='2008-11-11'
```

这样我们就不会有结果了！这是因为查询的日期不包含时间部分。

> **提示：**如果您想使查询更加简单和易于维护，请不要使用日期中的时间部分！
