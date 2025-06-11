---
layout: wiki  # 使用wiki布局模板
wiki: sql # 这是项目id，对应 /_data/wiki/sql.yml
title: SQL UPPER()函数
order: 14
---

## SQL UPPER()函数

------

SQL upper()函数字母大小写转换函数，将字母转成大写 - 返回字符串str，根据当前字符集映射的所有字符更改为大写。

### UPPER(str)

返回字符串str，根据当前字符集映射的所有字符更改为大写。

```
SQL> SELECT UPPER('Allah-hus-ngrok'); +---------------------------------------------------------+ | UPPER('Allah-hus-ngrok') | +---------------------------------------------------------+ | ALLAH-HUS-ngrok | +---------------------------------------------------------+ 1 row in set (0.00 sec)
```

