---
layout: wiki  # 使用wiki布局模板
wiki: redis # 这是项目id，对应 /_data/wiki/git.yml
title: Redis 备份和恢复
order: 1
---

# redis备份和恢复

SAVE 命令用于创建当前 Redis 数据库的备份。此命令将通过执行同步 SAVE 在 Redis 目录中创建 dump.rdb 文件。

**语法**

SAVE

**返回值**

执行成功后，SAVE 命令返回 OK。

------



## Redis备份示例

使用 SAVE 命令创建当前数据库的备份。

SAVE

![Redis备份1](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-backup1-1.png)

它将在 Redis 目录中创建 dump.rdb 文件。

可以看到 dump.rdb 文件已创建。

![Redis备份2](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-backup2-1.png)



## 还原Redis数据

将 Redis 备份文件（dump.rdb）移动到 Redis 目录中并启动服务器以恢复 Redis 数据。

查找 Redis 的安装目录，使用 Redis 的 CONFIG 命令，如下所示。

![Redis备份3](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-backup3-1.png)

Redis 服务器安装在以下目录中。

“/var/lib/redis”

------



## BGSAVE命令

BGSAVE 是创建 Redis 备份的备用命令。

此命令将启动备份过程并在后台运行。

**语法**

BGSAVE

**例**

![Redis备份4](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/redis-backup4-1.png)
