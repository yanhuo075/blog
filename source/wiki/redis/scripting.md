---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 脚本
order: 13
---

# Redis脚本

Redis 脚本使用 Lua 解释器来执行脚本。

自版本 2.6.0 开始内嵌于 Redis 中。

用于编写脚本的命令是 EVAL。

**句法**

```
redis 127.0.0.1:6379> EVAL script numkeys key [key ...] arg [arg ...]  
```

**例**

让我们举一个例子来看看 Redis 脚本的工作原理：

```
redis 127.0.0.1:6379> EVAL "return " 2 key1 key2 first second    
1) "key1"   
2) "key2"   
3) "first"   
4) "second"
```
