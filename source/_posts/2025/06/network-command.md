---
title: 华为交换机救命命令、断网不慌！
categories: [运维]
tags: [网络运维]
---

# 华为交换机救命命令、断网不慌！

> 当交换机端口闪红、网络突然卡死，哪些命令能救命？本文精选华为交换机运维**20条高频实战命令**，每条均附**真实命令格式+模拟输出结果**。从端口速查（`dis int bri`）到日志分析（`dis log`）、从安全审计到故障回退，手把手教你用命令窗口快速锁定问题，化被动救火为主动防御。

![](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250626152915_36.png)

#### 一、基础状态速查

- 1.**查看端口状态汇总**

  ```
  display interface brief
  ```

  ```
  GE1/0/1    UP      1G      full    # 正常
  GE1/0/2    DOWN    1G      full    # 异常！需排查
  ```

- 2.**检查CPU负载**

  ```
  display cpu-usage
  ```

  ```
  Slot 1: 28% (5s) 35% (1m)  # 持续>70%需警惕
  ```

- 3.**验证VLAN端口分布**

  ```
  display vlan 10
  ```

  ```
  VLAN 10: Ports-GE1/0/1(U) GE1/0/3(U)  # U=正常端口
  ```

- 4.**查看设备基本信息**

  ```
  display device
  ```

  ```
  Slot 1: S5735-H48UM4CC  Power:ON  # 硬件状态正常
  ```

- 5.**检查IP接口状态**

  ```
  display ip interface brief
  ```

  ```
  Vlanif10    192.168.10.1/24   UP    # 网关接口状态
  ```

#### 二、故障定位神器

- 6.**实时抓取系统日志**

  ```
  display logbuffer
  ```

  ```
  10:25:18 %LINK-3-UPDOWN: GE1/0/2 DOWN  # 端口异常事件
  ```

- 7.**追踪IP-MAC地址**

  ```
  display arp 192.168.1.10
  ```

  ```
  IP:192.168.1.10  MAC:5489-98d1-0101  Port:GE1/0/1  # 找到设备位置
  ```

- 8.**路由连通性测试**

  ```
  ping 8.8.8.8 -c 5
  ```

  ```
  !!! Reply from 8.8.8.1: bytes=56 time=15ms  # 全部通：!!!
  ```

- 9.**定位网络断点**

  ```
  tracert 8.8.8.8
  ```

  ```
  1  192.168.1.1  1ms  
  2  10.1.1.1     * * *   # 此处断网！
  ```

- 10.**查看路由表**

  ```
  display ip routing-table
  ```

  ```
  192.168.10.0/24  Direct  Vlanif10  # 直连路由存在
  ```

![图片](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/640)

#### 三、安全与维护

- 11.**保存当前配置（防丢失）**

  ```
  save
  ```

  ```
  Are you sure? [Y/N]: Y  # 必须确认！
  ```

- 12.**重置故障端口**

  ```
  interface GE1/0/2
   shutdown  # 关闭
   undo shutdown  # 重启
  ```

- 13.**查看SSH登录者**

  ```
  display ssh user-information
  ```

  ```
  User:admin  IP:10.1.1.25  Login:2025-06-15 09:30  # 监控非法访问
  ```

- 14.**清除ARP缓存（防欺骗）**

  ```
  reset arp all  # 危险操作！需断业务
  ```

- 15.**检查内存使用**

  ```
  display memory-usage
  ```

  ```
  Memory Usage: 56%  # >80%需优化
  ```

#### 四、高级诊断

- 16.**检查生成树状态**

  ```
  display stp brief
  ```

  ```
  GE1/0/1  ROOT    FORWARDING  # 根端口正常
  GE1/0/2  ALTE    DISCARDING  # 被阻塞端口
  ```

- 17.**查看MAC地址表**

  ```
  display mac-address | include 5489-98d1  # 过滤设备
  ```

  ```
  5489-98d1-0101  GE1/0/1  VLAN 10  # 定位接入端口
  ```

- 18.**测试端口环回（物理层诊断）**

  ```
  loopback internal  # 在端口视图下执行
  ```

  ```
  ! 若端口UP且收包正常 → 硬件无故障
  ```

- 19.**检查聚合链路**

  ```
  display eth-trunk 1
  ```

  ```
  GE1/0/3: Selected  # 聚合成员正常
  GE1/0/4: Unselected  # 异常成员
  ```

#### 五、救命操作

- 20.**恢复出厂设置（终极手段）**

  ```
  reset saved-configuration  # 清空配置
  reboot  # 重启生效
  ```

#### 运维铁律

- **每日必查**：`display interface brief` + `display cpu-usage` + `display memory-usage`
- **故障三板斧**：`display logbuffer` → `ping` → `tracert`
- **改配置后**：立即 `save`，重大变更前备份
- **慎用**：`reset` / `reboot` 需多人确认！

> 掌握这20条命令，相当于获得华为交换机的“听诊器”。日常巡检时多用`display`组合拳（端口/CPU/内存），故障时按**日志→连通性→路径追踪**三板斧推进。切记：**改配置必`save`，动核心需备份**。命令的价值在于组合应用——将碎片化操作转化为系统排障流程，方能在运维战场上游刃有余。
