---
layout: wiki  # 使用wiki布局模板
wiki: nightingale # 这是项目id，对应 /_data/wiki/nightingale.yml
title: 1.2 夜莺对比 Prometheus
order: 1
---

# 夜莺对比 Prometheus



夜莺监控（Nightingale）类似 Grafana，可以对接多种数据源，最常用的数据源就是 Prometheus（其他兼容 Prometheus 接口的数据源，比如 VictoriaMetrics、Thanos、M3DB，都可以看做是 Prometheus 类型），所以二者关系密切。

如果您有如下诉求，可以考虑夜莺：

- 有多套时序库，比如 Prometheus、VictoriaMetrics 等，想要使用一套统一的平台来管理各类告警规则，并有权限管控
- Prometheus 的告警引擎是单点的，担心单机挂掉导致告警引擎无法工作
- 除了 Prometheus 的告警，还需要 ElasticSearch、Loki、ClickHouse 等其他数据源的告警
- 需要更灵活的告警规则配置，比如控制生效时间、事件 Relabel、事件联动 CMDB、支持告警联动自愈脚本

夜莺监控也具备类似 Grafana 的可视化能力，不过没有 Grafana 道行深，以笔者观察来看，很多公司是一套组合方案（成年人的世界，没有非黑即白，都要）：

- 数据采集：组合使用了各种 agent 和 exporter，比如主要使用 Categraf（尤其是机器监控，和夜莺丝滑对接），辅以各类 Exporter
- 存储：时序库主要使用 VictoriaMetrics，因为 VictoriaMetrics 兼容 Prometheus，而且性能更好且有集群版本，对大部分公司，单机版就足够用了
- 告警引擎：使用夜莺，方便不同的团队管理协作，内置了一些规则开箱即用，告警规则的配置非常灵活，事件 Pipeline 机制方便和自己的 CMDB 等打通
- 看图可视化：使用 Grafana，图表更为炫酷，社区非常庞大，从 Grafana 站点可以找到很多别人做好的仪表盘，较为省心
- 告警事件 On-call 分发：使用 [FlashDuty](https://flashcat.cloud/product/flashduty/)，支持对接 Zabbix、Prometheus、夜莺、各云监控、Elastalert 等各类监控系统，收拢告警事件到一个平台，统一收敛降噪、排班、认领升级、响应、派发等。
