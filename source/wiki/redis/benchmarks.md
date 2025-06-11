---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 性能测试
order: 4
---

# Redis 基准性能测试

Redis 基准测试 redis-benchmark 是一种实用工具，用于通过同时使用 multiple(n) 命令来检查 Redis 的性能。

**语法**

```
redis-benchmark [option] [option value]
```



### 例

调用 Redis Benchmark 命令：

```
redis-benchmark -n 100000
```

**注意**：该命令是在 redis 的安装目录下执行的，而不是 redis 客户端的内部指令。

![Redis性能测试](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-benchmarks1-1.png)\



### redis 性能测试工具可选参数如下所示：

| 选项  | 描述                                       | 默认值    |
| :---- | :----------------------------------------- | :-------- |
| -h    | 指定服务器主机名                           | 127.0.0.1 |
| -p    | 指定服务器端口                             | 6379      |
| -s    | 指定服务器 socket                          |           |
| -c    | 指定并发连接数                             | 50        |
| -n    | 指定请求数                                 | 10000     |
| -d    | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| -k    | 1=keep alive 0=reconnect                   | 1         |
| -r    | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| -P    | 通过管道传输 <numreq> 请求                 | 1         |
| -q    | 强制退出 redis。仅显示 query/sec 值        |           |
| --csv | 以 CSV 格式输出                            |           |
| -l    | 生成循环，永久执行测试                     |           |
| -t    | 仅运行以逗号分隔的测试命令列表             |           |
| -I    | Idle 模式。仅打开 N 个 idle 连接并等待     |           |



### 实例

以下实例我们使用了多个参数来测试 redis 性能：主机为 127.0.0.1，端口号为 6379，执行的命令为 set,lpush，请求数为 10000，通过 -q 参数让结果只显示每秒执行的请求数。

```
redis-benchmark -h 127.0.0.1 -p 6379 -t set,lpush -n 100000 -q
```

![Redis基准3](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-benchmarks3-1.png)
