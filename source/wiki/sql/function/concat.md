---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL CONCAT()函数
order: 25
---

## SQL CONCAT 函数

------

**　CONCAT** 函数用于将两个字符串连接为一个字符串，试一下下面这个例子：

```
    SQL> SELECT CONCAT('FIRST ', 'SECOND');
    +----------------------------+
    | CONCAT('FIRST ', 'SECOND') |
    +----------------------------+
    | FIRST SECOND               |
    +----------------------------+
    1 row in set (0.00 sec)
```

要对 **CONCAT** 函数有更为深入的了解，请考虑 **employee_tbl** 表，表中记录如下所示：

```
    SQL> SELECT * FROM employee_tbl;
    +------+------+------------+--------------------+
    | id   | name | work_date  | daily_typing_pages |
    +------+------+------------+--------------------+
    |    1 | John | 2007-01-24 |                250 |
    |    2 | Ram  | 2007-05-27 |                220 |
    |    3 | Jack | 2007-05-06 |                170 |
    |    3 | Jack | 2007-04-06 |                100 |
    |    4 | Jill | 2007-04-06 |                220 |
    |    5 | Zara | 2007-06-06 |                300 |
    |    5 | Zara | 2007-02-06 |                350 |
    +------+------+------------+--------------------+
    7 rows in set (0.00 sec)
```

现在，假设你想要将上表中所有的姓名（name）、id和工作日（work_date）连接在一起，那么可以通过如下的命令来达到目的：

```
    SQL> SELECT CONCAT(id, name, work_date)
        -> FROM employee_tbl;
    +-----------------------------+
    | CONCAT(id, name, work_date) |
    +-----------------------------+
    | 1John2007-01-24             |
    | 2Ram2007-05-27              |
    | 3Jack2007-05-06             |
    | 3Jack2007-04-06             |
    | 4Jill2007-04-06             |
    | 5Zara2007-06-06             |
    | 5Zara2007-02-06             |
    +-----------------------------+
    7 rows in set (0.00 sec)
```