---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL 各种数据库的数据类型
order: 56
---

## SQL 各种数据库的数据类型

------

Microsoft Access、MySQL 和 SQL Server 所使用的数据类型和范围。

## Microsoft Access 数据类型

------

| 数据类型      | 描述                                                         | 存储     |
| :------------ | :----------------------------------------------------------- | :------- |
| Text          | 用于文本或文本与数字的组合。最多 255 个字符。                |          |
| Memo          | Memo 用于更大数量的文本。最多存储 65,536 个字符。**注释：**无法对 memo 字段进行排序。不过它们是可搜索的。 |          |
| Byte          | 允许 0 到 255 的数字。                                       | 1 字节   |
| Integer       | 允许介于 -32,768 与 32,767 之间的全部数字。                  | 2 字节   |
| Long          | 允许介于 -2,147,483,648 与 2,147,483,647 之间的全部数字。    | 4 字节   |
| Single        | 单精度浮点。处理大多数小数。                                 | 4 字节   |
| Double        | 双精度浮点。处理大多数小数。                                 | 8 字节   |
| Currency      | 用于货币。支持 15 位的元，外加 4 位小数。**提示：**您可以选择使用哪个国家的货币。 | 8 字节   |
| AutoNumber    | AutoNumber 字段自动为每条记录分配数字，通常从 1 开始。       | 4 字节   |
| Date/Time     | 用于日期和时间                                               | 8 字节   |
| Yes/No        | 逻辑字段，可以显示为 Yes/No、True/False 或 On/Off。在代码中，使用常量 True 和 False （等价于 1 和 0）。**注释：**Yes/No 字段中不允许 Null 值 | 1 比特   |
| Ole Object    | 可以存储图片、音频、视频或其他 BLOBs（Binary Large OBjects）。 | 最多 1GB |
| Hyperlink     | 包含指向其他文件的链接，包括网页。                           |          |
| Lookup Wizard | 允许您创建一个可从下拉列表中进行选择的选项列表。             | 4 字节   |



## MySQL 数据类型

------

在 MySQL 中，有三种主要的类型：Text（文本）、Number（数字）和 Date/Time（日期/时间）类型。

**　Text 类型：**

| 数据类型         | 描述                                                         |
| :--------------- | :----------------------------------------------------------- |
| CHAR(size)       | 保存固定长度的字符串（可包含字母、数字以及特殊字符）。在括号中指定字符串的长度。最多 255 个字符。 |
| VARCHAR(size)    | 保存可变长度的字符串（可包含字母、数字以及特殊字符）。在括号中指定字符串的最大长度。最多 255 个字符。**注释：**如果值的长度大于 255，则被转换为 TEXT 类型。 |
| TINYTEXT         | 存放最大长度为 255 个字符的字符串。                          |
| TEXT             | 存放最大长度为 65,535 个字符的字符串。                       |
| BLOB             | 用于 BLOBs（Binary Large OBjects）。存放最多 65,535 字节的数据。 |
| MEDIUMTEXT       | 存放最大长度为 16,777,215 个字符的字符串。                   |
| MEDIUMBLOB       | 用于 BLOBs（Binary Large OBjects）。存放最多 16,777,215 字节的数据。 |
| LONGTEXT         | 存放最大长度为 4,294,967,295 个字符的字符串。                |
| LONGBLOB         | 用于 BLOBs (Binary Large OBjects)。存放最多 4,294,967,295 字节的数据。 |
| ENUM(x,y,z,etc.) | 允许您输入可能值的列表。可以在 ENUM 列表中列出最大 65535 个值。如果列表中不存在插入的值，则插入空值。 **注释：**这些值是按照您输入的顺序排序的。 可以按照此格式输入可能的值： ENUM('X','Y','Z') |
| SET              | 与 ENUM 类似，不同的是，SET 最多只能包含 64 个列表项且 SET 可存储一个以上的选择。 |



**　Number 类型：**

| 数据类型        | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| TINYINT(size)   | -128 到 127 常规。0 到 255 无符号*。在括号中规定最大位数。   |
| SMALLINT(size)  | -32768 到 32767 常规。0 到 65535 无符号*。在括号中规定最大位数。 |
| MEDIUMINT(size) | -8388608 到 8388607 普通。0 to 16777215 无符号*。在括号中规定最大位数。 |
| INT(size)       | -2147483648 到 2147483647 常规。0 到 4294967295 无符号*。在括号中规定最大位数。 |
| BIGINT(size)    | -9223372036854775808 到 9223372036854775807 常规。0 到 18446744073709551615 无符号*。在括号中规定最大位数。 |
| FLOAT(size,d)   | 带有浮动小数点的小数字。在 size 参数中规定最大位数。在 d 参数中规定小数点右侧的最大位数。 |
| DOUBLE(size,d)  | 带有浮动小数点的大数字。在 size 参数中规定最大位数。在 d 参数中规定小数点右侧的最大位数。 |
| DECIMAL(size,d) | 作为字符串存储的 DOUBLE 类型，允许固定的小数点。在 size 参数中规定最大位数。在 d 参数中规定小数点右侧的最大位数。 |

> **注意：**这些整数类型拥有额外的选项 UNSIGNED。通常，整数可以是负数或正数。如果添加 UNSIGNED 属性，那么范围将从 0 开始，而不是某个负数。



**　Date 类型：**

| 数据类型    | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| DATE()      | 日期。格式：YYYY-MM-DD **注释：**支持的范围是从 '1000-01-01' 到 '9999-12-31' |
| DATETIME()  | *日期和时间的组合。格式：YYYY-MM-DD HH:MM:SS **注释：**支持的范围是从 '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59' |
| TIMESTAMP() | *时间戳。TIMESTAMP 值使用 Unix 纪元('1970-01-01 00:00:00' UTC) 至今的秒数来存储。格式：YYYY-MM-DD HH:MM:SS **注释：**支持的范围是从 '1970-01-01 00:00:01' UTC 到 '2038-01-09 03:14:07' UTC |
| TIME()      | 时间。格式：HH:MM:SS **注释：**支持的范围是从 '-838:59:59' 到 '838:59:59' |
| YEAR()      | 2 位或 4 位格式的年。 **注释：**4 位格式所允许的值：1901 到 2155。2 位格式所允许的值：70 到 69，表示从 1970 到 2069。 |

> **注意：**即便 DATETIME 和 TIMESTAMP 返回相同的格式，它们的工作方式很不同。在 INSERT 或 UPDATE 查询中，TIMESTAMP 自动把自身设置为当前的日期和时间。TIMESTAMP 也接受不同的格式，比如 YYYYMMDDHHMMSS、YYMMDDHHMMSS、YYYYMMDD 或 YYMMDD。



## SQL Server 数据类型

------

**　String 类型：**

| 数据类型       | 描述                                                 | 存储                      |
| :------------- | :--------------------------------------------------- | :------------------------ |
| char(n)        | 固定长度的字符串。最多 8,000 个字符。                | Defined width             |
| varchar(n)     | 可变长度的字符串。最多 8,000 个字符。                | 2 bytes + number of chars |
| varchar(max)   | 可变长度的字符串。最多 1,073,741,824 个字符。        | 2 bytes + number of chars |
| text           | 可变长度的字符串。最多 2GB 文本数据。                | 4 bytes + number of chars |
| nchar          | 固定长度的 Unicode 字符串。最多 4,000 个字符。       | Defined width x 2         |
| nvarchar       | 可变长度的 Unicode 字符串。最多 4,000 个字符。       |                           |
| nvarchar(max)  | 可变长度的 Unicode 字符串。最多 536,870,912 个字符。 |                           |
| ntext          | 可变长度的 Unicode 字符串。最多 2GB 文本数据。       |                           |
| bit            | 允许 0、1 或 NULL                                    |                           |
| binary(n)      | 固定长度的二进制字符串。最多 8,000 字节。            |                           |
| varbinary      | 可变长度的二进制字符串。最多 8,000 字节。            |                           |
| varbinary(max) | 可变长度的二进制字符串。最多 2GB。                   |                           |
| image          | 可变长度的二进制字符串。最多 2GB。                   |                           |



**　Number 类型：**

| 数据类型     | 描述                                                         | 存储        |
| :----------- | :----------------------------------------------------------- | :---------- |
| tinyint      | 允许从 0 到 255 的所有数字。                                 | 1 字节      |
| smallint     | 允许介于 -32,768 与 32,767 的所有数字。                      | 2 字节      |
| int          | 允许介于 -2,147,483,648 与 2,147,483,647 的所有数字。        | 4 字节      |
| bigint       | 允许介于 -9,223,372,036,854,775,808 与 9,223,372,036,854,775,807 之间的所有数字。 | 8 字节      |
| decimal(p,s) | 固定精度和比例的数字。 允许从 -10^38 +1 到 10^38 -1 之间的数字。 p 参数指示可以存储的最大位数（小数点左侧和右侧）。p 必须是 1 到 38 之间的值。默认是 18。 s 参数指示小数点右侧存储的最大位数。s 必须是 0 到 p 之间的值。默认是 0。 | 5-17 字节   |
| numeric(p,s) | 固定精度和比例的数字。 允许从 -10^38 +1 到 10^38 -1 之间的数字。 p 参数指示可以存储的最大位数（小数点左侧和右侧）。p 必须是 1 到 38 之间的值。默认是 18。 s 参数指示小数点右侧存储的最大位数。s 必须是 0 到 p 之间的值。默认是 0。 | 5-17 字节   |
| smallmoney   | 介于 -214,748.3648 与 214,748.3647 之间的货币数据。          | 4 字节      |
| money        | 介于 -922,337,203,685,477.5808 与 922,337,203,685,477.5807 之间的货币数据。 | 8 字节      |
| float(n)     | 从 -1.79E + 308 到 1.79E + 308 的浮动精度数字数据。 n 参数指示该字段保存 4 字节还是 8 字节。float(24) 保存 4 字节，而 float(53) 保存 8 字节。n 的默认值是 53。 | 4 或 8 字节 |
| real         | 从 -3.40E + 38 到 3.40E + 38 的浮动精度数字数据。            | 4 字节      |



**　Date 类型：**

| 数据类型       | 描述                                                         | 存储      |
| :------------- | :----------------------------------------------------------- | :-------- |
| datetime       | 从 1753 年 1 月 1 日 到 9999 年 12 月 31 日，精度为 3.33 毫秒。 | 8 字节    |
| datetime2      | 从 1753 年 1 月 1 日 到 9999 年 12 月 31 日，精度为 100 纳秒。 | 6-8 字节  |
| smalldatetime  | 从 1900 年 1 月 1 日 到 2079 年 6 月 6 日，精度为 1 分钟。   | 4 字节    |
| date           | 仅存储日期。从 0001 年 1 月 1 日 到 9999 年 12 月 31 日。    | 3 bytes   |
| time           | 仅存储时间。精度为 100 纳秒。                                | 3-5 字节  |
| datetimeoffset | 与 datetime2 相同，外加时区偏移。                            | 8-10 字节 |
| timestamp      | 存储唯一的数字，每当创建或修改某行时，该数字会更新。timestamp 值基于内部时钟，不对应真实时间。每个表只能有一个 timestamp 变量。 |           |



**　其他数据类型：**

| 数据类型         | 描述                                                         |
| :--------------- | :----------------------------------------------------------- |
| sql_variant      | 存储最多 8,000 字节不同数据类型的数据，除了 text、ntext 以及 timestamp。 |
| uniqueidentifier | 存储全局唯一标识符 (GUID)。                                  |
| xml              | 存储 XML 格式化数据。最多 2GB。                              |
| cursor           | 存储对用于数据库操作的指针的引用。                           |
| table            | 存储结果集，供稍后处理。                                     |
