---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 7.6 sqlite
order: 5
---

# sqlite

sqlite是一款非常流行的轻型数据库，在python环境下可以直接使用。在手机端，许多app的数据就存储在sqlite中。相比于mysql，oracle, postgresql等数据库，我最喜欢它的地方在于，使用sqlite无需复杂的安装和配置。

## 1、 python连接sqlite

只需两行代码，就可以连接到sqlite数据库

```python
import sqlite3
conn = sqlite3.connect('test.db')
```

如果test.db这个文件不存在，则上面的代码会创建它并连接，如果存在则直接连接。

除了创建文件外，还可以在内存中创建数据

```python
import sqlite3
conn = sqlite3.connect(":memory:")
```

## 2、创建表

下面的代码将创建一个张user表

```python
import sqlite3
conn = sqlite3.connect('test.db')
cursor = conn.cursor()

table_sql = """
create table user(
  id INTEGER PRIMARY KEY autoincrement NOT NULL ,
  name text NOT NULL,
  age INTEGER NOT NULL
)
"""
cursor.execute(table_sql)
conn.commit()  # 一定要提交,否则不会执行sql
conn.close()
```

sqlite一共有5中数据类型可以定义

| 类型    | 描述                                                      |
| ------- | --------------------------------------------------------- |
| NULL    | NULL 值                                                   |
| INTEGER | 带符号的整数,根据值的大小存储在 1、2、3、4、6 或 8 字节中 |
| REAL    | 浮点值，存储为 8 字节的 IEEE 浮点数字。                   |
| TEXT    | 字符串，使用数据库编码（UTF-8、UTF-16BE 或 UTF-16LE）存储 |
| BLOB    | blob 数据，完全根据它的输入存储                           |

## 3. insert数据

```python
import sqlite3
conn = sqlite3.connect('test.db')
cursor = conn.cursor()

sql_lst = [
    "insert into user(name, age)values('lili', 18)",
    "insert into user(name, age)values('poly', 19)",
    "insert into user(name, age)values('lilei', 30)"
]
for sql in sql_lst:
    cursor.execute(sql)
    conn.commit()
conn.close()
```

## 4. 查询

先看一个简单的示例

```python
import sqlite3
conn = sqlite3.connect('test.db')
conn.row_factory = sqlite3.Row
cursor = conn.cursor()

sql = "select * from user"
cursor.execute(sql)
rows = cursor.fetchall()  # 获取全部数据
for row in rows:
    print(row.keys(), tuple(row))

conn.close()
```

运行结果

```text
['id', 'name', 'age'] (1, 'lili', 18)
['id', 'name', 'age'] (2, 'poly', 19)
['id', 'name', 'age'] (3, 'lilei', 30)
```

- row.keys() 返回列的名字
- tuple(row) 获取tuple形式的数据

如果想以字典形式获取数据，则需要指定工厂方法，也就是一个解析数据的函数，将元组类型数据转换为字典类型数据。

```python
import sqlite3
conn = sqlite3.connect('test.db')

def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d

conn.row_factory = dict_factory
cursor = conn.cursor()

sql = "select * from user"
cursor.execute(sql)
rows = cursor.fetchall()  # 获取全部数据
for row in rows:
    print(row)

conn.close()
```

## 4. update数据

```python
import sqlite3
conn = sqlite3.connect('test.db')

def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d

conn.row_factory = dict_factory
cursor = conn.cursor()

# 修改数据
update_sql = "update user set age = 22 where id = 1"
cursor.execute(update_sql)
conn.commit()

sql = "select * from user"
cursor.execute(sql)
rows = cursor.fetchall()  # 获取全部数据
for row in rows:
    print(row)

conn.close()
```

## 5. 删除

```python
import sqlite3
conn = sqlite3.connect('test.db')

def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d

conn.row_factory = dict_factory
cursor = conn.cursor()

# 删除数据
delete_sql = "delete from user where id = 1"
cursor.execute(delete_sql)
conn.commit()

sql = "select * from user"
cursor.execute(sql)
rows = cursor.fetchall()  # 获取全部数据
for row in rows:
    print(row)

conn.close()
```

## 6. 批量插入, executemany

如果你有大量数据需要写入，那么不建议你使用execute，因为每一次执行execute，都要和数据库进行一次数据交互，而批量执行则可以免去这种频繁的数据交互。

```python
import sqlite3
conn = sqlite3.connect('test.db')

def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d

conn.row_factory = dict_factory
cursor = conn.cursor()

# 批量执行
sql = "insert into user(name, age)values(?, ?)"
user_lst = [('lili', 18), ('poly', 19), ('lilei', 30)]
cursor.executemany(sql, user_lst)

sql = "select * from user"
cursor.execute(sql)
rows = cursor.fetchall()  # 获取全部数据
for row in rows:
    print(row)

conn.close()
```
