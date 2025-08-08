---
layout: wiki  # 使用wiki布局模板
wiki: nightingale # 这是项目id，对应 /_data/wiki/nightingale.yml
title: 3.4 Datadog-Agent
order: 3
---

# Datadog-Agent



## 配置 

Datadog-agent 的配置文件路径为 `/etc/datadog-agent/datadog.yaml`，修改配置文件中的 `dd_url` 项。

```yaml
dd_url: http://nightingale-address/datadog
```

`nightingale-address` 为你的夜莺地址。

## 重启 

重启 Datadog-Agent。

```bash
systemctl restart datadog-agent
```
