layout: wiki  # 使用wiki布局模板
wiki: prometheus # 这是项目id，对应 /_data/wiki/prometheus.yml
title: 5.3 常用Exporter
order: 27
---

# 常用Exporter

在第1章中，我们已经初步了解了Node Exporter的使用场景和方法。本小节，将会介绍更多常用的Exporter用法。包括如何监控容器运行状态，如何监控和评估MySQL服务的运行状态以及如何通过Prometheus实现基于网络探测的黑盒监控。



# 容器监控：cAdvisor

Docker是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux/Windows/Mac机器上。容器镜像正成为一个新的标准化软件交付方式。

例如，可以通过以下命令快速在本地启动一个Nginx服务：

```shell
docker run -itd nginx
```

为了能够获取到Docker容器的运行状态，用户可以通过Docker的stats命令获取到当前主机上运行容器的统计信息，可以查看容器的CPU利用率、内存使用量、网络IO总量以及磁盘IO总量等信息。

```shell
$ docker stats
CONTAINER           CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
9a1648bec3b2        0.30%               196KiB / 3.855GiB     0.00%               828B / 0B           827kB / 0B          1
```

除了使用命令以外，用户还可以通过Docker提供的HTTP API查看容器详细的监控统计信息。

## 使用CAdvisor

CAdvisor是Google开源的一款用于展示和分析容器运行状态的可视化工具。通过在主机上运行CAdvisor用户可以轻松的获取到当前主机上容器的运行统计信息，并以图表的形式向用户展示。

在本地运行CAdvisor也非常简单，直接运行一下命令即可：

```shell
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```

通过访问[http://localhost:8080](http://localhost:8080/)可以查看，当前主机上容器的运行状态，如下所示：

![CAdvisor可视化：CPU总量](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250807175730813.png)CAdvisor可视化：CPU总量

CAdvisor是一个简单易用的工具，相比于使用Docker命令行工具，用户不用再登录到服务器中即可以可视化图表的形式查看主机上所有容器的运行状态。

而在多主机的情况下，在所有节点上运行一个CAdvisor再通过各自的UI查看监控信息显然不太方便，同时CAdvisor默认只保存2分钟的监控数据。好消息是CAdvisor已经内置了对Prometheus的支持。访问http://localhost:8080/metrics即可获取到标准的Prometheus监控样本输出:

```text
# HELP cadvisor_version_info A metric with a constant '1' value labeled by kernel version, OS version, docker version, cadvisor version & cadvisor revision.
# TYPE cadvisor_version_info gauge
cadvisor_version_info{cadvisorRevision="1e567c2",cadvisorVersion="v0.28.3",dockerVersion="17.09.1-ce",kernelVersion="4.9.49-moby",osVersion="Alpine Linux v3.4"} 1
# HELP container_cpu_load_average_10s Value of container cpu load average over the last 10 seconds.
# TYPE container_cpu_load_average_10s gauge
container_cpu_load_average_10s{container_label_maintainer="",id="/",image="",name=""} 0
container_cpu_load_average_10s{container_label_maintainer="",id="/docker",image="",name=""} 0
container_cpu_load_average_10s{container_label_maintainer="",id="/docker/15535a1e09b3a307b46d90400423d5b262ec84dc55b91ca9e7dd886f4f764ab3",image="busybox",name="lucid_shaw"} 0
container_cpu_load_average_10s{container_label_maintainer="",id="/docker/46750749b97bae47921d49dccdf9011b503e954312b8cffdec6268c249afa2dd",image="google/cadvisor:latest",name="cadvisor"} 0
container_cpu_load_average_10s{container_label_maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>",id="/docker/f51fd4d4f410965d3a0fd7e9f3250218911c1505e12960fb6dd7b889e75fc114",image="nginx",name="confident_brattain"} 0
```

下面表格中列举了一些CAdvisor中获取到的典型监控指标：

| 指标名称                               | 类型    | 含义                                         |
| -------------------------------------- | ------- | -------------------------------------------- |
| container_cpu_load_average_10s         | gauge   | 过去10秒容器CPU的平均负载                    |
| container_cpu_usage_seconds_total      | counter | 容器在每个CPU内核上的累积占用时间 (单位：秒) |
| container_cpu_system_seconds_total     | counter | System CPU累积占用时间（单位：秒）           |
| container_cpu_user_seconds_total       | counter | User CPU累积占用时间（单位：秒）             |
| container_fs_usage_bytes               | gauge   | 容器中文件系统的使用量(单位：字节)           |
| container_fs_limit_bytes               | gauge   | 容器可以使用的文件系统总量(单位：字节)       |
| container_fs_reads_bytes_total         | counter | 容器累积读取数据的总量(单位：字节)           |
| container_fs_writes_bytes_total        | counter | 容器累积写入数据的总量(单位：字节)           |
| container_memory_max_usage_bytes       | gauge   | 容器的最大内存使用量（单位：字节）           |
| container_memory_usage_bytes           | gauge   | 容器当前的内存使用量（单位：字节             |
| container_spec_memory_limit_bytes      | gauge   | 容器的内存使用量限制                         |
| machine_memory_bytes                   | gauge   | 当前主机的内存总量                           |
| container_network_receive_bytes_total  | counter | 容器网络累积接收数据总量（单位：字节）       |
| container_network_transmit_bytes_total | counter | 容器网络累积传输数据总量（单位：字节）       |

## 与Prometheus集成

修改/etc/prometheus/prometheus.yml，将cAdvisor添加监控数据采集任务目标当中：

```yaml
- job_name: cadvisor
  static_configs:
  - targets:
    - localhost:8080
```

启动Prometheus服务:

```shell
prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/data/prometheus
```

启动完成后，可以在Prometheus UI中查看到当前所有的Target状态：

![Target](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250807175805629.png)Target

当能够正常采集到cAdvisor的样本数据后，可以通过以下表达式计算容器的CPU使用率：

```text
sum(irate(container_cpu_usage_seconds_total{image!=""}[1m])) without (cpu)
```

![容器CPU使用率](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250807175851158.png)容器CPU使用率

查询容器内存使用量（单位：字节）:

```text
container_memory_usage_bytes{image!=""}
```

查询容器网络接收量速率（单位：字节/秒）：

```text
sum(rate(container_network_receive_bytes_total{image!=""}[1m])) without (interface)
```

![容器网络接收量 字节/秒](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250807175926434.png)容器网络接收量 字节/秒

查询容器网络传输量速率（单位：字节/秒）：

```text
sum(rate(container_network_transmit_bytes_total{image!=""}[1m])) without (interface)
```

![容器网络传输量 字节/秒](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250807180002958.png)容器网络传输量 字节/秒

查询容器文件系统读取速率（单位：字节/秒）：

```text
sum(rate(container_fs_reads_bytes_total{image!=""}[1m])) without (device)
```

![容器文件系统读取速率 字节/秒](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250807180041592.png)容器文件系统读取速率 字节/秒

查询容器文件系统写入速率（单位：字节/秒）：

```text
sum(rate(container_fs_writes_bytes_total{image!=""}[1m])) without (device)
```

![容器文件系统写入速率 字节/秒](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250807180123081.png)



# 监控MySQL运行状态：MySQLD Exporter

MySQL是一个关系型数据库管理系统，由瑞典MySQL AB公司开发，目前属于Oracle旗下的产品。 MySQL是最流行的关系型数据库管理系统之一。数据库的稳定运行是保证业务可用性的关键因素之一。这一小节当中将介绍如何使用Prometheus提供的MySQLD Exporter实现对MySQL数据库性能以及资源利用率的监控和度量。

## 二进制部署MySQL_Exporter

```bash
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.12.0/mysqld_exporter-0.12.0.linux-amd64.tar.gz
tar xvf mysqld_exporter-0.12.0.linux-amd64.tar.gz
mv mysqld_exporter-0.12.0.linux-amd64 /data/
# 创建配置文件
cat >> /data/mysql_exporter/localhost_db.cnf <<EOF
[client]
user=mysqld_exporter
password=12345678
EOF
# 创建systemd服务
cat > /etc/systemd/system/mysql_exporter.service << EOF
[Unit]
Description=mysql_exporter
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/data/mysql_exporter/mysqld_exporter --config.my-cnf="/data/mysql_exporter/localhost_db.cnf" --web.listen-address=":9105"
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF
```

## grant permission to database.

```mysql
GRANT REPLICATION CLIENT, PROCESS ON *.* TO 'mysqld_exporter'@'localhost' identified by '12345678';
GRANT SELECT ON performance_schema.* TO 'mysqld_exporter'@'localhost';
flush privileges;
```

## Docker部署部署MySQLD Exporter

为了简化测试环境复杂度，这里使用Docker Compose定义并启动MySQL以及MySQLD Exporter：

```
version: '3'
services:
  mysql:
    image: mysql:5.7
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=database
  mysqlexporter:
    image: prom/mysqld-exporter
    ports:
      - "9104:9104"
    environment:
      - DATA_SOURCE_NAME=root:password@(mysql:3306)/database
```

这里通过环境变量DATA_SOURCE_NAME方式定义监控目标。使用Docker Compose启动测试用的MySQL实例以及MySQLD Exporter:

```
$ docker-compose up -d
```

启动完成后，可以通过以下命令登录到MySQL容器当中，并执行MySQL相关的指令：

```
$ docker exec -it <mysql_container_id> mysql -uroot -ppassword
mysql>
```

可以通过[http://localhost:9104](http://localhost:9104/)访问MySQLD Exporter暴露的服务：

可以通过/metrics查看mysql_up指标判断当前MySQLD Exporter是否正常连接到了MySQL实例，当指标值为1时表示能够正常获取监控数据：

```
# HELP mysql_up Whether the MySQL server is up.
# TYPE mysql_up gauge
mysql_up 1
```

修改Prometheus配置文件/etc/prometheus/prometheus.yml，增加对MySQLD Exporter实例的采集任务配置:

```
- job_name: mysqld
  static_configs:
  - targets:
    - localhost:9104
```

启动Prometheus:

```
prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/data/prometheus
```

通过Prometheus的状态页，可以查看当前Target的状态：

![MySQLD Exporter实例状态](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250807180357819.png)MySQLD Exporter实例状态

为了确保数据库的稳定运行，通常会关注一下四个与性能和资源利用率相关的指标：查询吞吐量、连接情况、缓冲池使用情况以及查询执行性能等。

## 监控数据库吞吐量

对于数据库而言，最重要的工作就是实现对数据的增、删、改、查。为了衡量数据库服务器当前的吞吐量变化情况。在MySQL内部通过一个名为Questions的计数器，当客户端发送一个查询语句后，其值就会+1。可以通过以下MySQL指令查询Questions等服务器状态变量的值：

```shell
mysql> SHOW GLOBAL STATUS LIKE "Questions";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Questions     | 1326  |
+---------------+-------+
1 row in set (0.00 sec)
```

MySQLD Exporter中返回的样本数据中通过mysql_global_status_questions反映当前Questions计数器的大小：

```text
# HELP mysql_global_status_questions Generic metric from SHOW GLOBAL STATUS.
# TYPE mysql_global_status_questions untyped
mysql_global_status_questions 1016
```

通过以下PromQL可以查看当前MySQL实例查询速率的变化情况，查询数量的突变往往暗示着可能发生了某些严重的问题，因此用于用户应该关注并且设置响应的告警规则，以及时获取该指标的变化情况：

```text
rate(mysql_global_status_questions[2m])
```

一般还可以从监控读操作和写操作的执行情况进行判断。通过MySQL全局状态中的Com_select可以查询到当前服务器执行查询语句的总次数：相应的，也可以通过Com_insert、Com_update以及Com_delete的总量衡量当前服务器写操作的总次数，例如，可以通过以下指令查询当前MySQL实例insert语句的执行次数总量：

```shell
mysql> SHOW GLOBAL STATUS LIKE "Com_insert";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Com_insert    | 0     |
+---------------+-------+
1 row in set (0.00 sec)
```

从MySQLD Exporter的/metrics返回的监控样本中，可以通过global_status_commands_total获取当前实例各类指令执行的次数：

```text
# HELP mysql_global_status_commands_total Total number of executed MySQL commands.
# TYPE mysql_global_status_commands_total counter
mysql_global_status_commands_total{command="admin_commands"} 0
mysql_global_status_commands_total{command="alter_db"} 0
mysql_global_status_commands_total{command="alter_db_upgrade"} 0
mysql_global_status_commands_total{command="select"} 10
mysql_global_status_commands_total{command="insert"} 2
mysql_global_status_commands_total{command="update"} 2
mysql_global_status_commands_total{command="delete"} 1
```

用户可以通过以下PromQL查看当前MySQL实例写操作速率的变化情况：

```text
sum(rate(mysql_global_status_commands_total{command=~"insert|update|delete"}[2m])) without (command)
```

## 连接情况

在MySQL中通过全局设置max_connections限制了当前服务器允许的最大客户端连接数量。一旦可用连接数被用尽，新的客户端连接都会被直接拒绝。 因此当监控MySQL运行状态时，需要时刻关注MySQL服务器的连接情况。用户可以通过以下指令查看当前MySQL服务的max_connections配置：

```shell
mysql> SHOW VARIABLES LIKE 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+
1 row in set (0.01 sec)
```

MySQL默认的最大链接数为151。临时调整最大连接数，可以通过以下指令进行设置：

```mysql
SET GLOBAL max_connections = 200;
```

如果想永久化设置，则需要通过修改MySQL配置文件my.cnf，添加以下内容：

```text
max_connections = 200
```

通过Global Status中的Threads_connected、Aborted_connects、Connection_errors_max_connections以及Threads_running可以查看当前MySQL实例的连接情况。

例如，通过以下指令可以直接当前MySQL实例的连接数：

```mysql
mysql> SHOW GLOBAL STATUS LIKE "Threads_connected";
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_connected | 1     |
+-------------------+-------+
1 row in set (0.00 sec)
```

当所有可用连接都被占用时，如果一个客户端尝试连接至MySQL，会出现“Too many connections(连接数过多)”错误，同时Connection_errors_max_connections的值也会增加。为了防止出现此类情况，你应该监控可用连接的数量，并确保其值保持在max_connections限制以内。同时如果Aborted_connects的数量不断增加时，说明客户端尝试连接到MySQL都失败了。此时可以通过Connection_errors_max_connections以及Connection_errors_internal分析连接失败的问题原因。

下面列举了与MySQL连接相关的监控指标：

- mysql_global_variables_max_connections： 允许的最大连接数；
- mysql_global_status_threads_connected： 当前开放的连接；
- mysql_global_status_threads_running：当前开放的连接；
- mysql_global_status_aborted_connects：当前开放的连接；
- mysql_global_status_connection_errors_total{error="max_connections"}：由于超出最大连接数导致的错误；
- mysql_global_status_connection_errors_total{error="internal"}：由于系统内部导致的错误；

通过PromQL查询当前剩余的可用连接数：

```text
mysql_global_variables_max_connections - mysql_global_status_threads_connected
```

使用PromQL查询当前MySQL实例连接拒绝数：

```text
mysql_global_status_aborted_connects
```

## 监控缓冲池使用情况

MySQL默认的存储引擎InnoDB使用了一片称为缓冲池的内存区域，用于缓存数据表以及索引的数据。 当缓冲池的资源使用超出限制后，可能会导致数据库性能的下降，同时很多查询命令会直接在磁盘中执行，导致磁盘I/O不断攀升。 因此，应该关注MySQL缓冲池的资源使用情况，并且在合理的时间扩大缓冲池的大小可以优化数据库的性能。

Innodb_buffer_pool_pages_total反映了当前缓冲池中的内存页的总页数。可以通过以下指令查看：

```text
mysql> SHOW GLOBAL STATUS LIKE "Innodb_buffer_pool_pages_total";
+--------------------------------+-------+
| Variable_name                  | Value |
+--------------------------------+-------+
| Innodb_buffer_pool_pages_total | 8191  |
+--------------------------------+-------+
1 row in set (0.02 sec)
```

MySQLD Exporter通过以下指标返回缓冲池中各类内存页的数量：

```text
# HELP mysql_global_status_buffer_pool_pages Innodb buffer pool pages by state.
# TYPE mysql_global_status_buffer_pool_pages gauge
mysql_global_status_buffer_pool_pages{state="data"} 516
mysql_global_status_buffer_pool_pages{state="dirty"} 0
mysql_global_status_buffer_pool_pages{state="free"} 7675
mysql_global_status_buffer_pool_pages{state="misc"} 0
```

Innodb_buffer_pool_read_requests记录了正常从缓冲池读取数据的请求数量。可以通过以下指令查看：

```text
mysql> SHOW GLOBAL STATUS LIKE "Innodb_buffer_pool_read_requests";
+----------------------------------+--------+
| Variable_name                    | Value  |
+----------------------------------+--------+
| Innodb_buffer_pool_read_requests | 797023 |
+----------------------------------+--------+
1 row in set (0.00 sec)
```

MySQLD Exporter通过以下指标返回缓冲池中Innodb_buffer_pool_read_requests的值：

```text
# HELP mysql_global_status_innodb_buffer_pool_read_requests Generic metric from SHOW GLOBAL STATUS.
# TYPE mysql_global_status_innodb_buffer_pool_read_requests untyped
mysql_global_status_innodb_buffer_pool_read_requests 736711
```

当缓冲池无法满足时，MySQL只能从磁盘中读取数据。Innodb_buffer_pool_reads即记录了从磁盘读取数据的请求数量。通常来说从内存中读取数据的速度要比从磁盘中读取快很多，因此，如果Innodb_buffer_pool_reads的值开始增加，可能意味着数据库的性能有问题。 可以通过以下只能查看Innodb_buffer_pool_reads的数量

```text
mysql> SHOW GLOBAL STATUS LIKE "Innodb_buffer_pool_reads";
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| Innodb_buffer_pool_reads | 443   |
+--------------------------+-------+
1 row in set (0.00 sec)
```

在MySQLD Exporter中可以通过以下指标查看Innodb_buffer_pool_reads的数量。

```text
# HELP mysql_global_status_innodb_buffer_pool_reads Generic metric from SHOW GLOBAL STATUS.
# TYPE mysql_global_status_innodb_buffer_pool_reads untyped
mysql_global_status_innodb_buffer_pool_reads 443
```

通过以上监控指标，以及实际监控的场景，我们可以利用PromQL快速建立多个监控项。

通过以下PromQL可以得到各个MySQL实例的缓冲池利用率。一般来说还需要结合Innodb_buffer_pool_reads的增长率情况来结合判断缓冲池大小是否合理：

```
(sum(mysql_global_status_buffer_pool_pages) by (instance) - sum(mysql_global_status_buffer_pool_pages{state="free"}) by (instance)) / sum(mysql_global_status_buffer_pool_pages) by (instance)
```

也可以通过以下PromQL计算2分钟内磁盘读取请求次数的增长率的变化情况：

```
rate(mysql_global_status_innodb_buffer_pool_reads[2m])
```

## 查询性能

MySQL还提供了一个Slow_queries的计数器，当查询的执行时间超过long_query_time的值后，计数器就会+1，其默认值为10秒，可以通过以下指令在MySQL中查询当前long_query_time的设置：

```
mysql> SHOW VARIABLES LIKE 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)
```

通过以下指令可以查看当前MySQL实例中Slow_queries的数量：

```
mysql> SHOW GLOBAL STATUS LIKE "Slow_queries";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 0     |
+---------------+-------+
1 row in set (0.00 sec)
```

MySQLD Exporter返回的样本数据中，通过以下指标展示当前的Slow_queries的值：

```
# HELP mysql_global_status_slow_queries Generic metric from SHOW GLOBAL STATUS.
# TYPE mysql_global_status_slow_queries untyped
mysql_global_status_slow_queries 0
```

通过监控Slow_queries的增长率，可以反映出当前MySQL服务器的性能状态，可以通过以下PromQL查询Slow_queries的增长情况：

```
rate(mysql_global_status_slow_queries[2m])
```

在MySQL中还可以通过安装response time插件，从而支持记录查询时间区间的统计信息。启动该功能后MySQLD Exporter也会自动获取到相关数据，从而可以细化MySQL查询响应时间的分布情况。 感兴趣的读者可以自行尝试。



# 网络探测：Blackbox Exporter

在本章的前几个小节中我们主要介绍了Prometheus下如何进行白盒监控，我们监控主机的资源用量、容器的运行状态、数据库中间件的运行数据。 这些都是支持业务和服务的基础设施，通过白盒能够了解其内部的实际运行状态，通过对监控指标的观察能够预判可能出现的问题，从而对潜在的不确定因素进行优化。而从完整的监控逻辑的角度，除了大量的应用白盒监控以外，还应该添加适当的黑盒监控。黑盒监控即以用户的身份测试服务的外部可见性，常见的黑盒监控包括HTTP探针、TCP探针等用于检测站点或者服务的可访问性，以及访问效率等。

黑盒监控相较于白盒监控最大的不同在于黑盒监控是以故障为导向当故障发生时，黑盒监控能快速发现故障，而白盒监控则侧重于主动发现或者预测潜在的问题。一个完善的监控目标是要能够从白盒的角度发现潜在问题，能够在黑盒的角度快速发现已经发生的问题。

![黑盒监控和白盒监控](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250807180512238.png)

​                                                                                            黑盒监控和白盒监控

## 使用Blackbox Exporter

Blackbox Exporter是Prometheus社区提供的官方黑盒监控解决方案，其允许用户通过：HTTP、HTTPS、DNS、TCP以及ICMP的方式对网络进行探测。用户可以直接使用go get命令获取Blackbox Exporter源码并生成本地可执行文件：

```
go get prometheus/blackbox_exporter
```

运行Blackbox Exporter时，需要用户提供探针的配置信息，这些配置信息可能是一些自定义的HTTP头信息，也可能是探测时需要的一些TSL配置，也可能是探针本身的验证行为。在Blackbox Exporter每一个探针配置称为一个module，并且以YAML配置文件的形式提供给Blackbox Exporter。 每一个module主要包含以下配置内容，包括探针类型（prober）、验证访问超时时间（timeout）、以及当前探针的具体配置项：

```
  # 探针类型：http、 tcp、 dns、 icmp.
  prober: <prober_string>

  # 超时时间
  [ timeout: <duration> ]

  # 探针的详细配置，最多只能配置其中的一个
  [ http: <http_probe> ]
  [ tcp: <tcp_probe> ]
  [ dns: <dns_probe> ]
  [ icmp: <icmp_probe> ]
```

下面是一个简化的探针配置文件blockbox.yml，包含两个HTTP探针配置项：

```
modules:
  http_2xx:
    prober: http
    http:
      method: GET
  http_post_2xx:
    prober: http
    http:
      method: POST
```

通过运行以下命令，并指定使用的探针配置文件启动Blockbox Exporter实例：

```
blackbox_exporter --config.file=/etc/prometheus/blackbox.yml
```

启动成功后，就可以通过访问http://127.0.0.1:9115/probe?module=http_2xx&target=baidu.com对baidu.com进行探测。这里通过在URL中提供module参数指定了当前使用的探针，target参数指定探测目标，探针的探测结果通过Metrics的形式返回：

```
# HELP probe_dns_lookup_time_seconds Returns the time taken for probe dns lookup in seconds
# TYPE probe_dns_lookup_time_seconds gauge
probe_dns_lookup_time_seconds 0.011633673
# HELP probe_duration_seconds Returns how long the probe took to complete in seconds
# TYPE probe_duration_seconds gauge
probe_duration_seconds 0.117332275
# HELP probe_failed_due_to_regex Indicates if probe failed due to regex
# TYPE probe_failed_due_to_regex gauge
probe_failed_due_to_regex 0
# HELP probe_http_content_length Length of http content response
# TYPE probe_http_content_length gauge
probe_http_content_length 81
# HELP probe_http_duration_seconds Duration of http request by phase, summed over all redirects
# TYPE probe_http_duration_seconds gauge
probe_http_duration_seconds{phase="connect"} 0.055551141
probe_http_duration_seconds{phase="processing"} 0.049736019
probe_http_duration_seconds{phase="resolve"} 0.011633673
probe_http_duration_seconds{phase="tls"} 0
probe_http_duration_seconds{phase="transfer"} 3.8919e-05
# HELP probe_http_redirects The number of redirects
# TYPE probe_http_redirects gauge
probe_http_redirects 0
# HELP probe_http_ssl Indicates if SSL was used for the final redirect
# TYPE probe_http_ssl gauge
probe_http_ssl 0
# HELP probe_http_status_code Response HTTP status code
# TYPE probe_http_status_code gauge
probe_http_status_code 200
# HELP probe_http_version Returns the version of HTTP of the probe response
# TYPE probe_http_version gauge
probe_http_version 1.1
# HELP probe_ip_protocol Specifies whether probe ip protocol is IP4 or IP6
# TYPE probe_ip_protocol gauge
probe_ip_protocol 4
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
```

从返回的样本中，用户可以获取站点的DNS解析耗时、站点响应时间、HTTP响应状态码等等和站点访问质量相关的监控指标，从而帮助管理员主动的发现故障和问题。

## 与Prometheus集成

接下来，只需要在Prometheus下配置对Blockbox Exporter实例的采集任务即可。最直观的配置方式：

```
- job_name: baidu_http2xx_probe
  params:
    module:
    - http_2xx
    target:
    - baidu.com
  metrics_path: /probe
  static_configs:
  - targets:
    - 127.0.0.1:9115
- job_name: prometheus_http2xx_probe
  params:
    module:
    - http_2xx
    target:
    - prometheus.io
  metrics_path: /probe
  static_configs:
  - targets:
    - 127.0.0.1:9115
```

这里分别配置了名为baidu_http2x_probe和prometheus_http2xx_probe的采集任务，并且通过params指定使用的探针（module）以及探测目标（target）。

那问题就来了，假如我们有N个目标站点且都需要M种探测方式，那么Prometheus中将包含N * M个采集任务，从配置管理的角度来说显然是不可接受的。 在第7章的“服务发现与Relabel”小节，我们介绍了Prometheus的Relabeling能力，这里我们也可以采用Relabling的方式对这些配置进行简化，配置方式如下：

```
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http.
        - https://prometheus.io   # Target to probe with https.
        - http://example.com:8080 # Target to probe with http on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115
```

这里针对每一个探针服务（如http_2xx）定义一个采集任务，并且直接将任务的采集目标定义为我们需要探测的站点。在采集样本数据之前通过relabel_configs对采集任务进行动态设置。

- 第1步，根据Target实例的地址，写入`__param_target`标签中。`__param_<name>`形式的标签表示，在采集任务时会在请求目标地址中添加`<name>`参数，等同于params的设置；
- 第2步，获取__param_target的值，并覆写到instance标签中；
- 第3步，覆写Target实例的`__address__`标签值为BlockBox Exporter实例的访问地址。

通过以上3个relabel步骤，即可大大简化Prometheus任务配置的复杂度:

![Blackbox Target实例](https://www.prometheus.wang/exporter/static/relabel_blackbox_targets.png)Blackbox Target实例

接下来，我们将详细介绍Blackbox中常用的HTTP探针使用方式

## HTTP探针

HTTP探针是进行黑盒监控时最常用的探针之一，通过HTTP探针能够网站或者HTTP服务建立有效的监控，包括其本身的可用性，以及用户体验相关的如响应时间等等。除了能够在服务出现异常的时候及时报警，还能帮助系统管理员分析和优化网站体验。

在上一小节讲过，Blockbox Exporter中所有的探针均是以Module的信息进行配置。如下所示，配置了一个最简单的HTTP探针：

```
modules:
  http_2xx_example:
    prober: http
    http:
```

通过prober配置项指定探针类型。配置项http用于自定义探针的探测方式，这里有没对http配置项添加任何配置，表示完全使用HTTP探针的默认配置，该探针将使用HTTP GET的方式对目标服务进行探测，并且验证返回状态码是否为2XX，是则表示验证成功，否则失败。

### 自定义HTTP请求

HTTP服务通常会以不同的形式对外展现，有些可能就是一些简单的网页，而有些则可能是一些基于REST的API服务。 对于不同类型的HTTP的探测需要管理员能够对HTTP探针的行为进行更多的自定义设置，包括：HTTP请求方法、HTTP头信息、请求参数等。对于某些启用了安全认证的服务还需要能够对HTTP探测设置相应的Auth支持。对于HTTPS类型的服务还需要能够对证书进行自定义设置。

如下所示，这里通过method定义了探测时使用的请求方法，对于一些需要请求参数的服务，还可以通过headers定义相关的请求头信息，使用body定义请求内容：

```
http_post_2xx:
    prober: http
    timeout: 5s
    http:
      method: POST
      headers:
        Content-Type: application/json
      body: '{}'
```

如果HTTP服务启用了安全认证，Blockbox Exporter内置了对basic_auth的支持，可以直接设置相关的认证信息即可：

```
http_basic_auth_example:
    prober: http
    timeout: 5s
    http:
      method: POST
      headers:
        Host: "login.example.com"
      basic_auth:
        username: "username"
        password: "mysecret"
```

对于使用了Bear Token的服务也可以通过bearer_token配置项直接指定令牌字符串，或者通过bearer_token_file指定令牌文件。

对于一些启用了HTTPS的服务，但是需要自定义证书的服务，可以通过tls_config指定相关的证书信息：

```
 http_custom_ca_example:
    prober: http
    http:
      method: GET
      tls_config:
        ca_file: "/certs/my_cert.crt"
```

### 自定义探针行为

在默认情况下HTTP探针只会对HTTP返回状态码进行校验，如果状态码为2XX（200 <= StatusCode < 300）则表示探测成功，并且探针返回的指标probe_success值为1。

如果用户需要指定HTTP返回状态码，或者对HTTP版本有特殊要求，如下所示，可以使用valid_http_versions和valid_status_codes进行定义：

```
  http_2xx_example:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2"]
      valid_status_codes: []
```

默认情况下，Blockbox返回的样本数据中也会包含指标probe_http_ssl，用于表明当前探针是否使用了SSL：

```
# HELP probe_http_ssl Indicates if SSL was used for the final redirect
# TYPE probe_http_ssl gauge
probe_http_ssl 0
```

而如果用户对于HTTP服务是否启用SSL有强制的标准。则可以使用fail_if_ssl和fail_if_not_ssl进行配置。fail_if_ssl为true时，表示如果站点启用了SSL则探针失败，反之成功。fail_if_not_ssl刚好相反。

```
  http_2xx_example:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: []
      method: GET
      no_follow_redirects: false
      fail_if_ssl: false
      fail_if_not_ssl: false
```

除了基于HTTP状态码，HTTP协议版本以及是否启用SSL作为控制探针探测行为成功与否的标准以外，还可以匹配HTTP服务的响应内容。使用fail_if_matches_regexp和fail_if_not_matches_regexp用户可以定义一组正则表达式，用于验证HTTP返回内容是否符合或者不符合正则表达式的内容。

```
  http_2xx_example:
    prober: http
    timeout: 5s
    http:
      method: GET
      fail_if_matches_regexp:
        - "Could not connect to database"
      fail_if_not_matches_regexp:
        - "Download the latest version here"
```

最后需要提醒的时，默认情况下HTTP探针会走IPV6的协议。 在大多数情况下，可以使用preferred_ip_protocol=ip4强制通过IPV4的方式进行探测。在Bloackbox响应的监控样本中，也会通过指标probe_ip_protocol，表明当前的协议使用情况：

```
# HELP probe_ip_protocol Specifies whether probe ip protocol is IP4 or IP6
# TYPE probe_ip_protocol gauge
probe_ip_protocol 6
```

除了支持对HTTP协议进行网络探测以外，Blackbox还支持对TCP、DNS、ICMP等其他网络协议，感兴趣的读者可以从Blackbox的Github项目中获取更多使用信息

#### https://github.com/prometheus/blackbox_exporter。



# 监控Redis运行状态：Redis Exporter

## 部署Redis Exporter

这里使用二进制部署启动Redis以及Redis Exporter，具体安装细节就不做重复了

```
mkdir /data/redis_exporter
sudo wget https://github.com/oliver006/redis_exporter/releases/download/v1.3.2/redis_exporter-v1.3.2.linux-amd64.tar.gz
tar -xvf  redis_exporter-v1.3.2.linux-amd64.tar.gz
mv redis_exporter-v1.3.2.linux-amd64 /data/redis_exporter

wget  https://grafana.com/api/dashboards/763/revisions/1/download

## 无密码
./redis_exporter redis//172.26.42.229:6379
## 有密码
./redis_exporter -redis.addr 172.26.42.229:6379  -redis.password 123456

cat > /etc/systemd/system/redis_exporter.service <<EOF

[Unit]
Description=redis_exporter
After=network.target

[Service]
Type=simple
User=prometheus
ExecStart=/data/redis_exporter/redis_exporter -redis.addr 172.26.42.229:6379  -redis.password 123456
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF


- job_name: redis
static_configs:
  - targets: ['1172.26.42.229:9121']
    labels:
      instance: redis120
```



