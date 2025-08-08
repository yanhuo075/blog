---
layout: wiki  # 使用wiki布局模板
wiki: nightingale # 这是项目id，对应 /_data/wiki/nightingale.yml
title: 5.6 Redis监控
order: 5
---

# Redis监控



> 对于监控系统，基础功能的强弱确实非常关键，但是如何在不同的场景落地实践，则更为关键。在《监控实践》章节，搜罗各类监控实践经验，会以不同的组件分门别类，您如果对某个组件有好的实践经验，欢迎提 PR，把您的文章链接附到对应的组件目录下。

- [Categraf 采集 Redis 监控数据](https://flashcat.cloud/docs/content/flashcat-monitor/categraf/plugin/redis/)
- [使用 Cprobe 监控 MySQL、Redis、MongoDB、Oracle、Postgres 等](https://mp.weixin.qq.com/s/6dEgijH-nWddbK8yCEUUOA)

## Redis 监控采集原理 

不管使用 Categraf 还是 Redis-Exporter 采集 Redis 的监控数据，原理都是类似的，通过 Redis 连接地址、用户名密码等信息连到 Redis 上，执行 `info` 之类的命令获取监控数据。

## 如何接入 Redis-Exporter 

有些用户用了 Categraf 采集机器指标、进程指标、自定义插件，但是没有使用 Categraf 采集 Redis 的监控数据，而是使用了 Redis-Exporter。然后就比较困惑：如何把 Redis-Exporter 采集的数据接入到夜莺中？

有两个办法：

- 直接在你的时序库里配置 Scrape 规则，抓取 Redis-Exporter 的数据
- 使用 Categraf 的 input.prometheus 插件，抓取 Redis-Exporter 的数据



## Redis 监控概述

Redis 的监控和 MySQL、MongoDB 等数据库的监控类似，采集器 agent（比如 Categraf、Redis Exporter） 作为 client 连到 Redis 实例上，执行一些命令获取监控数据，比如连到 Redis 上执行 `info` 命令。

## Categraf 监控 Redis 快速入门

Categraf 内置各类监控插件，也内置了 Redis 的监控插件，其配置文件在 Categraf 的 `conf/input.redis/redis.toml`，下面是一个极简配置内容：

```toml
[[instances]]
address = "127.0.0.1:6379"
```

上例配置极为简单，就是给出了 Redis 实例的连接地址（前提是 Redis 没有设置认证信息）。然后通过 Categraf 的 `--test` 命令可以测试配置是否正确：

```shell
./categraf --test --inputs redis
```

结果输出如下：

```shell
ulric@ulric-flashcat categraf % ./categraf.mac --test --inputs redis
2024/11/22 08:27:11 main.go:149: I! runner.binarydir: /Users/ulric/works/gopath/src/categraf
2024/11/22 08:27:11 main.go:150: I! runner.hostname: ulric-flashcat.local
2024/11/22 08:27:11 main.go:151: I! runner.fd_limits: (soft=61440, hard=unlimited)
2024/11/22 08:27:11 main.go:152: I! runner.vm_limits: (soft=unlimited, hard=unlimited)
...
1732235231 08:27:11 redis_ping_use_seconds address=127.0.0.1:6379 agent_hostname=mac-ulric-flashcat region=ulric 0.007866
1732235231 08:27:11 redis_up address=127.0.0.1:6379 agent_hostname=mac-ulric-flashcat region=ulric 1
1732235231 08:27:11 redis_cmdstat_calls address=127.0.0.1:6379 agent_hostname=mac-ulric-flashcat command=del region=ulric replica_role=master 2
...
```

篇幅所限，上面只贴了 3 个指标，实际会采集更多指标。只要看到超过 3 个指标输出，就说明 Redis 监控配置正确。

## 在夜莺里查看仪表盘

![Redis内置仪表盘](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250808180335776.png)

菜单入口：`集成中心-模板中心-搜索Redis-仪表盘`，可以看到夜莺（我所用的版本是 7.7.1）内置了 Redis 的仪表盘，可以直接使用。

![Redis仪表盘](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250808180420463.png)

上图是 `"Redis by address"` 的仪表盘效果。当然了，我们不推荐你直接查看内置仪表盘，因为有些仪表盘只能导入到自己的业务组下才能查看。而且内置仪表盘通常不能修改，把内置仪表盘导入自己的业务组下，然后您可以根据自己的需求，随意修改。

## Redis 两种采集方式

和 MySQL 类似，Redis 也有两种采集方式。一种是使用一个 Categraf 采集所有的 Redis，一对多的方式。另一种是每个 Categraf 只负责自己所在机器上的 Redis，一对一的方式。

### 一对多的方式

此方式尤其适合采集云上 Redis，因为云上 Redis 所在的宿主我们无法直接部署 Categraf。但是此种方式受网络影响较大。此种方式下，redis.toml 里配置很多 Redis 实例的目标地址，其目标地址肯定是不同的，所以在仪表盘里可以通过 `address` 来区分不同的 Redis 实例。

### 一对一的方式

Categraf 只负责本地 Redis 的采集，此种方式受网络影响较小。此种方式下，redis.toml 里通常配置的是 `127.0.0.1:6379`，所以通过 address 标签无法区分不同的 Redis 实例。此时有两个办法来区分不同的 Redis 实例：

- 使用 ident 标签，即要查看 Redis 数据的时候，先找到对应的宿主机器，再查看该机器上的 Redis 实例
- 使用 instance 标签，需要在 redis.toml 里手工配置上 instance 标签，比如 `instance = "my-redis"`，然后在仪表盘里通过 instance 来区分不同的 Redis 实例。

```toml
[[instances]]
address = "127.0.0.1:6379"
labels = { instance="n9e-redis" }
```

## 如何配置采集多个 Redis 实例

一对多的方式，显然是需要在 redis.toml 中配置多个 Redis 实例的目标地址。一对一的方式，有时也需要配置多个，因为在大内存的机器上，可能会有多个 Redis 实例。配置方式也简单，就是把 `[[instances]]` 配置段复制多份即可。

```toml
[[instances]]
address = "127.0.0.1:6379"
labels = { instance="n9e-redis-01" }

[[instances]]
address = "127.0.0.1:6380"
labels = { instance="n9e-redis-02" }
```

## Redis 集群监控

Redis 集群监控，本质上还是监控每个 Redis 实例。只是 Redis 集群的每个实例有一定的逻辑属性，都属于某个集群，所以在监控时，可以通过 `cluster` 标签来区分不同的 Redis 集群。

```toml
[[instances]]
address = "127.0.0.1:6379"
labels = { instance="n9e-redis-01", cluster="n9e" }

[[instances]]
address = "127.0.0.1:6380"
labels = { instance="n9e-redis-02", cluster="n9e" }
```

上例配置了两个 Redis 实例，都属于 `n9e` 集群。在仪表盘里，可以通过 `cluster` 标签来区分不同的 Redis 集群。夜莺里还没有内置按照 cluster 标签来区分 Redis 集群的仪表盘，需要自己创建。

## Redis 插件配置详解

下面我们把 Categraf 的 Redis 采集插件的整个配置文件贴到下面，方便大家查阅：

```toml
## 采集频率，单位是秒
# interval = 15

## Redis 实例配置，直到出现下一个 [[instances]] 为止
## 如果没有下一个 [[instances]]，则到文件末尾为止
[[instances]]
## Redis 实例地址
address = "127.0.0.1:6379"
## Redis 实例认证信息
# username = ""
# password = ""
## Categraf 采集 Redis 时的 client 的连接池大小
# pool_size = 2

## 是否开启slowlog 收集，因为 slowlog 中的 cmd 指令会放到标签中，这无疑会大幅增加 time series 的数量
## 而且这些 time series 具备极高的流失率，对时序库的性能会造成影响，所以开启此功能要慎重
# gather_slowlog = true

## 最多收集少条slowlog
# slowlog_max_len = 100

## 收集距离现在多少秒以内的slowlog
## 注意插件的采集周期,该参数不要小于采集周期，否则会有slowlog查不到
# slowlog_time_window=30

## slowlog 收集到的指标样例，cmd 标签中的值是 slowlog 中的命令
## redis_slow_log{ident=dev-01 client_addr=127.0.0.1:56364 client_name= cmd="info ALL" log_id=983} 74 (单位微秒)

## 自定义命令收集，根据用户指定的命令收集指标，命令执行结果需要时数字，或可以转换为数字的字符串
## 因为在 Prometheus 生态里，指标值必须是数字。指标名字由用户自定义，通过下面的 metric 字段指定
# commands = [
#     {command = ["get", "sample-key1"], metric = "custom_metric_name1"},
#     {command = ["get", "sample-key2"], metric = "custom_metric_name2"}
# ]

## 采集频率的倍数，即最终采集频率是 interval * interval_times
# interval_times = 1

## 自定义标签。这里的标签会附加到采集到的所有指标上
# labels = { instance="n9e-10.2.3.4:6379" }

## TLS 相关配置
## use_tls 设置为 true 时，下面的其他配置才生效
# use_tls = false
# tls_min_version = "1.2"
# tls_ca = "/etc/categraf/ca.pem"
# tls_cert = "/etc/categraf/cert.pem"
# tls_key = "/etc/categraf/key.pem"
## Use TLS but skip chain & host verification
# insecure_skip_verify = true
```

## 总结

本文讲解了 Redis 监控插件的配置，对于有 Redis 监控、Redis 集群监控需求的朋友，希望上面的内容可以帮到您。
