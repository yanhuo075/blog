---
layout: wiki  # 使用wiki布局模板
wiki: docker # 这是项目id，对应 /_data/wiki/docker.yml
title: 3. Docker 服务安装
order: 2
---

## 3. Docker 服务安装

### 3.1 CentOS系统

##### 3.1.1 参考官方文档地址：

```bash
https://docs.docker.com/engine/install/centos/
```

##### 3.1.2 如果存在旧版docker请先卸载

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

##### 3.1.3 设置docker yum源（二选一）

设置为阿里云的源速度可以快一点（推荐）

```bash
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

如果不想阿里云的源，也可用官方源（可能遇到网络问题）

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

##### 3.1.4 安装docker

```bash
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

##### 3.1. 5 启动docker

```bash
sudo systemctl start docker
```

##### 3.1.6 设置开机自启动

```bash
sudo systemctl enable docker
```

##### 3.1.7 查看版本

```bash
docker -v
docker info
```



### 3.2 Ubuntu系统

在Ubuntu 20.04上使用国内源安装Docker，可以使用清华大学源或阿里云源，具体如下。

##### 3.2.1 先更新软件包，安装备要apt软件

```csharp
# 更新软件包索引
sudo apt-get update
 
# 安装需要的软件包以使apt能够通过HTTPS使用仓库
sudo apt-get install ca-certificates curl gnupg lsb-release
```

##### 3.2.2 选择安装源

- 使用清化大学源

```bash
# 添加Docker官方的GPG密钥
curl -fsSL https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
 
# 设置稳定版仓库
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

- 使用阿里云源

```csharp
# 添加阿里云官方GPG密钥
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
 
# 写入阿里云Docker仓库地址
sudo sh -c 'echo "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list'
```

##### 3.2.3 更新源并安装Docker

```sql
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# 验证是否成功安装了docker
sudo systemctl status docker
docker --version
```

##### 3.2.4 修改docker的``配置文件

修改docker的`/etc/docker/daemon.json`配置文件，如果在不存在则手动创建，文件内容如下：

```bash
# 修改daemon.json文件，
vim /etc/docker/daemon.json

# daemon.json内容如下：
{
    "registry-mirrors": [
        "https://dockerproxy.com",
        "https://docker.m.daocloud.io",
        "https://cr.console.aliyun.com",
        "https://ccr.ccs.tencentyun.com",
        "https://hub-mirror.c.163.com",
        "https://mirror.baidubce.com",
        "https://docker.nju.edu.cn",
        "https://docker.mirrors.sjtug.sjtu.edu.cn",
        "https://github.com/ustclug/mirrorrequest",
        "https://registry.docker-cn.com"
    ]
}
```

##### 3.2.5 重载配置文件，并重启 docker
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```



##### 3.2.6 查看 Registry Mirrors 配置是否成功
```
sudo docker info 
```

