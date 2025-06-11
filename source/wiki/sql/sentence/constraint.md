---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL Constraint约束
order: 41
---

SQL约束用于指定表中数据的规则。

## SQL 约束

------

约束是作用于数据表中列上的规则，用于限制表中数据的类型。约束的存在保证了数据库中数据的精确性和可靠性。

约束有列级和表级之分，列级约束作用于单一的列，而表级约束作用于整张数据表。

下面是 SQL 中常用的约束，这些约束虽然已经在关系型数据库管理系统一章中讨论过了，但是仍然值得在这里回顾一遍。

- ：保证列中数据不能有 NULL 值
- ：提供该列数据未指定时所采用的默认值
- ：保证列中的所有数据各不相同
- ：唯一标识数据表中的行/记录
- ：唯一标识其他表中的一条行/记录
- ：此约束保证列中的所有值满足某一条件
- ：用于在数据库中快速创建或检索数据

约束可以在创建表时规定（通过 CREATE TABLE 语句），或者在表创建之后规定（通过 ALTER TABLE 语句）。

## SQL创建约束

------

当使用CREATE TABLE语句创建表时，或者在使用ALTER TABLE语句创建表之后，可以指定约束。

语法

```
CREATE TABLE table_name (
    column1 datatype constraint,
    column2 datatype constraint,
    column3 datatype constraint,
    ....
);
```

### SQL CREATE TABLE + CONSTRAINT 语法

```
CREATE TABLE table_name                
(                
column_name1 data_type(size) constraint_name,                
column_name2 data_type(size) constraint_name,                
column_name3 data_type(size) constraint_name,                
....                
);
```

## 删除约束

------

任何现有约束都可以通过在 ALTER TABLE 命令中指定 DROP CONSTRAINT 选项的方法删除掉。

例如，要去除 EMPLOYEES 表中的主键约束，可以使用下述命令：

```
ALTER TABLE EMPLOYEES DROP CONSTRAINT EMPLOYEES_PK;
```

一些数据库实现可能提供了删除特定约束的快捷方法。例如，要在 Oracle 中删除一张表的主键约束，可以使用如下命令：

```
ALTER TABLE EMPLOYEES DROP PRIMARY KEY;
```

某些数据库实现允许禁用约束。这样与其从数据库中永久删除约束，你可以只是临时禁用掉它，过一段时间后再重新启用。

## 完整性约束

------

完整性约束用于保证关系型数据库中数据的精确性和一致性。对于关系型数据库来说，数据完整性由参照完整性（referential integrity，RI）来保证。

有很多种约束可以起到参照完整性的作用，这些约束包括主键约束（Primary Key）、外键约束（Foreign Key）、唯一性约束（Unique Constraint）以及上面提到的其他约束。



## SQL NOT NULL 约束

------

在默认的情况下，表的列接受 NULL 值。

NOT NULL 约束强制列不接受 NULL 值。

NOT NULL 约束强制字段始终包含值。这意味着，如果不向字段添加值，就无法插入新记录或者更新记录。

下面的 SQL 强制 "P_Id" 列和 "LastName" 列不接受 NULL 值：

```
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```



## SQL UNIQUE 约束

------

UNIQUE 约束唯一标识数据库表中的每条记录。

UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。

PRIMARY KEY 约束拥有自动定义的 UNIQUE 约束。

请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

## CREATE TABLE 时的 SQL UNIQUE 约束

------

下面的 SQL 在 "Persons" 表创建时在 "P_Id" 列上创建 UNIQUE 约束：

**　MySQL：**

```
CREATE TABLE Persons                
(                
P_Id int NOT NULL,                
LastName varchar(255) NOT NULL,                
FirstName varchar(255),                
Address varchar(255),                
City varchar(255),                
UNIQUE (P_Id)                
)
```

**　SQL Server / Oracle / MS Access：**

```
CREATE TABLE Persons                
(                
P_Id int NOT NULL UNIQUE,                
LastName varchar(255) NOT NULL,                
FirstName varchar(255),                
Address varchar(255),                
City varchar(255)                
)
```

如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：

**　MySQL / SQL Server / Oracle / MS Access：**

```
CREATE TABLE Persons                
(                
P_Id int NOT NULL,                
LastName varchar(255) NOT NULL,                
FirstName varchar(255),                
Address varchar(255),                
City varchar(255),                
CONSTRAINT uc_PersonID UNIQUE (P_Id,LastName)                
)
```

## ALTER TABLE 时的 SQL UNIQUE 约束

------

当表已被创建时，如需在 "P_Id" 列创建 UNIQUE 约束，请使用下面的 SQL：

**　MySQL / SQL Server / Oracle / MS Access：**

```
ALTER TABLE Persons                
ADD UNIQUE (P_Id)
```

如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：

**　MySQL / SQL Server / Oracle / MS Access：**

```
ALTER TABLE Persons                
ADD CONSTRAINT uc_PersonID UNIQUE (P_Id,LastName)
```

## 撤销 UNIQUE 约束

------

如需撤销 UNIQUE 约束，请使用下面的 SQL：

**　MySQL：**

```
ALTER TABLE Persons                
DROP INDEX uc_PersonID
```

**　SQL Server / Oracle / MS Access：**

```
ALTER TABLE Persons                
DROP CONSTRAINT uc_PersonID
```



## SQL PRIMARY KEY 约束

------

PRIMARY KEY 约束唯一标识数据库表中的每条记录。

主键必须包含唯一的值。

主键列不能包含 NULL 值。

每个表都应该有一个主键，并且每个表只能有一个主键。

## CREATE TABLE 时的 SQL PRIMARY KEY 约束rimary-key-约束)

------

下面的 SQL 在 "Persons" 表创建时在 "P_Id" 列上创建 PRIMARY KEY 约束：

**　MySQL：**

```
CREATE TABLE Persons                
(                
P_Id int NOT NULL,                
LastName varchar(255) NOT NULL,                
FirstName varchar(255),                
Address varchar(255),                
City varchar(255),                
PRIMARY KEY (P_Id)                
)
```

**　SQL Server / Oracle / MS Access：**

```
CREATE TABLE Persons                
(                
P_Id int NOT NULL PRIMARY KEY,                
LastName varchar(255) NOT NULL,                
FirstName varchar(255),                
Address varchar(255),                
City varchar(255)                
)
```

如需命名 PRIMARY KEY 约束，并定义多个列的 PRIMARY KEY 约束，请使用下面的 SQL 语法：

**　MySQL / SQL Server / Oracle / MS Access：**

```
CREATE TABLE Persons                
(                
P_Id int NOT NULL,                
LastName varchar(255) NOT NULL,                
FirstName varchar(255),                
Address varchar(255),                
City varchar(255),                
CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName)                
)
```

**　注释：** 在上面的实例中，只有一个主键 PRIMARY KEY（pk_PersonID）。然而，pk_PersonID 的值是由两个列（P_Id 和 LastName）组成的。

## ALTER TABLE 时的 SQL PRIMARY KEY 约束

------

当表已被创建时，如需在 "P_Id" 列创建 PRIMARY KEY 约束，请使用下面的 SQL：

**　MySQL / SQL Server / Oracle / MS Access：**

sql

```
ALTER TABLE Persons                
ADD PRIMARY KEY (P_Id)
```

如需命名 PRIMARY KEY 约束，并定义多个列的 PRIMARY KEY 约束，请使用下面的 SQL 语法：

**　MySQL / SQL Server / Oracle / MS Access：**

```
ALTER TABLE Persons                
ADD CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName)
```

**　注释：**如果您使用 ALTER TABLE 语句添加主键，必须把主键列声明为不包含 NULL 值（在表首次创建时）。

## 撤销 PRIMARY KEY 约束

------

如需撤销 PRIMARY KEY 约束，请使用下面的 SQL：

**　MySQL：**

```
ALTER TABLE Persons                
DROP PRIMARY KEY
```

**　SQL Server / Oracle / MS Access：**

```
ALTER TABLE Persons               
DROP CONSTRAINT pk_PersonID
```



## SQL FOREIGN KEY 约束

------

一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY。

让我们通过一个实例来解释外键。请看下面两个表：

"Persons" 表：

| P_Id | LastName  | FirstName | Address      | City      |
| :--- | :-------- | :-------- | :----------- | :-------- |
| 1    | Hansen    | Ola       | Timoteivn 10 | Sandnes   |
| 2    | Svendson  | Tove      | Borgvn 23    | Sandnes   |
| 3    | Pettersen | Kari      | Storgt 20    | Stavanger |

"Orders" 表：

| O_Id | OrderNo | P_Id |
| :--- | :------ | :--- |
| 1    | 77895   | 3    |
| 2    | 44678   | 3    |
| 3    | 22456   | 2    |
| 4    | 24562   | 1    |

请注意，"Orders" 表中的 "P_Id" 列指向 "Persons" 表中的 "P_Id" 列。

"Persons" 表中的 "P_Id" 列是 "Persons" 表中的 PRIMARY KEY。

"Orders" 表中的 "P_Id" 列是 "Orders" 表中的 FOREIGN KEY。

FOREIGN KEY 约束用于预防破坏表之间连接的行为。

FOREIGN KEY 约束也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。

## CREATE TABLE 时的 SQL FOREIGN KEY 约束reign-key-约束)

------

下面的 SQL 在 "Orders" 表创建时在 "P_Id" 列上创建 FOREIGN KEY 约束：

**　MySQL：**

```
CREATE TABLE Orders                
(                
O_Id int NOT NULL,                
OrderNo int NOT NULL,                
P_Id int,               
PRIMARY KEY (O_Id),                
FOREIGN KEY (P_Id) REFERENCES Persons(P_Id)                
)
```

**　SQL Server / Oracle / MS Access：**

```
CREATE TABLE Orders                
(                
O_Id int NOT NULL PRIMARY KEY,                
OrderNo int NOT NULL,                
P_Id int FOREIGN KEY REFERENCES Persons(P_Id)                
)
```

如需命名 FOREIGN KEY 约束，并定义多个列的 FOREIGN KEY 约束，请使用下面的 SQL 语法：

**　MySQL / SQL Server / Oracle / MS Access：**

```
CREATE TABLE Orders                
(                
O_Id int NOT NULL,                
OrderNo int NOT NULL,                
P_Id int,                
PRIMARY KEY (O_Id),                
CONSTRAINT fk_PerOrders FOREIGN KEY (P_Id)                
REFERENCES Persons(P_Id)                
)
```

## ALTER TABLE 时的 SQL FOREIGN KEY 约束

------

当 "Orders" 表已被创建时，如需在 "P_Id" 列创建 FOREIGN KEY 约束，请使用下面的 SQL：

**　MySQL / SQL Server / Oracle / MS Access：**

```
ALTER TABLE Orders                
ADD FOREIGN KEY (P_Id)                
REFERENCES Persons(P_Id)
```

如需命名 FOREIGN KEY 约束，并定义多个列的 FOREIGN KEY 约束，请使用下面的 SQL 语法：

**　MySQL / SQL Server / Oracle / MS Access：**

```
ALTER TABLE Orders                
ADD CONSTRAINT fk_PerOrders                
FOREIGN KEY (P_Id)                
REFERENCES Persons(P_Id)
```

## 撤销 FOREIGN KEY 约束

------

如需撤销 FOREIGN KEY 约束，请使用下面的 SQL：

**　MySQL：**

```
ALTER TABLE Orders                
DROP FOREIGN KEY fk_PerOrders
```

**　SQL Server / Oracle / MS Access：**

```
ALTER TABLE Orders                
DROP CONSTRAINT fk_PerOrders
```



## SQL DEFAULT 约束

------

DEFAULT 约束用于向列中插入默认值。

如果没有规定其他的值，那么会将默认值添加到所有的新记录。

## CREATE TABLE 时的 SQL DEFAULT 约束

------

下面的 SQL 在 "Persons" 表创建时在 "City" 列上创建 DEFAULT 约束：

**　My SQL / SQL Server / Oracle / MS Access：**

```
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255) DEFAULT 'Sandnes'
)
```

通过使用类似 GETDATE() 这样的函数，DEFAULT 约束也可以用于插入系统值：

```
CREATE TABLE Orders
(
O_Id int NOT NULL,
OrderNo int NOT NULL,
P_Id int,
OrderDate date DEFAULT GETDATE()
)
```

## ALTER TABLE 时的 SQL DEFAULT 约束

------

当表已被创建时，如需在 "City" 列创建 DEFAULT 约束，请使用下面的 SQL：

**　MySQL：**

```
ALTER TABLE Persons
ALTER City SET DEFAULT 'SANDNES'
```

**　SQL Server / MS Access：**

```
ALTER TABLE Persons 
ADD CONSTRAINT DF_Persons_City DEFAULT('SANDNES') FOR City

--注释
--Persons 为表名
--City 为列名
--DF_Persons_City 为我们创建的默认约束的名称 约束名称一般为:约束类型简称_表名_列名
```

**　Oracle：**

```
ALTER TABLE Persons
MODIFY City DEFAULT 'SANDNES'
```

## 撤销 DEFAULT 约束

------

如需撤销 DEFAULT 约束，请使用下面的 SQL：

**　MySQL：**

```
ALTER TABLE Persons
ALTER City DROP DEFAULT
```

**　SQL Server / Oracle / MS Access：**

```
ALTER TABLE Persons
ALTER COLUMN City DROP DEFAULT
```



## SQL CHECK 约束

------

CHECK 约束用于限制列中的值的范围。

如果对单个列定义 CHECK 约束，那么该列只允许特定的值。

如果对一个表定义 CHECK 约束，那么此约束会基于行中其他列的值在特定的列中对值进行限制。

## CREATE TABLE 时的 SQL CHECK 约束

------

下面的 SQL 在 "Persons" 表创建时在 "P_Id" 列上创建 CHECK 约束。CHECK 约束规定 "P_Id" 列必须只包含大于 0 的整数。

**　MySQL：**

```
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CHECK (P_Id>0)
)
```

**　SQL Server / Oracle / MS Access：**

```
CREATE TABLE Persons
(
P_Id int NOT NULL CHECK (P_Id>0),
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

如需命名 CHECK 约束，并定义多个列的 CHECK 约束，请使用下面的 SQL 语法：

**　MySQL / SQL Server / Oracle / MS Access：**

```
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes')
)
```

## ALTER TABLE 时的 SQL CHECK 约束

当表已被创建时，如需在 "P_Id" 列创建 CHECK 约束，请使用下面的 SQL：

**　MySQL / SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
ADD CHECK (P_Id>0)
```

如需命名 CHECK 约束，并定义多个列的 CHECK 约束，请使用下面的 SQL 语法：

**　MySQL / SQL Server / Oracle / MS Access：**

```
ALTER TABLE Persons
ADD CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes')
```

## 撤销 CHECK 约束

------

如需撤销 CHECK 约束，请使用下面的 SQL：

**　SQL Server / Oracle / MS Access：**

```
ALTER TABLE Persons
DROP CONSTRAINT chk_Person
```

**　MySQL：**

```
ALTER TABLE Persons
DROP CHECK chk_Person
```
