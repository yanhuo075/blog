---
layout: wiki  # 使用wiki布局模板
wiki: docker # 这是项目id，对应 /_data/wiki/docker.yml
title: 11. DockerHub 国内加速
order: 10
---

## 11. DockerHub 国内加速

### 11.1 DockerHub是什么

Docker Hub是 Docker 提供的一项服务，用于与您的团队查找和共享容器映像。 它是世界上最大的容器映像存储库，其中包含一系列内容源，包括容器社区开发人员，开源项目和独立软件供应商（ISV），它们在容器中构建和分发其代码。

[![国内DockerHub镜像加速](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/68747470733a2f2f62322e77776b656a697368652e746f702f57502d43444e2d30322f323032322f3230323230383034313134313238372e77656270)](https://camo.githubusercontent.com/a87b8b1fa28c6f25ce8c18df0e0527f9705ea4e05f8773212665588bc229dc87/68747470733a2f2f62322e77776b656a697368652e746f702f57502d43444e2d30322f323032322f3230323230383034313134313238372e77656270)

**DockerHub镜像加速器为啥不能拉取访问了？**

6月6日，[上海交大的 Docker Hub 镜像加速器](https://mirrors.ustc.edu.cn/help/dockerhub.html)宣布因监管要求被下架。[具体可看此通知](https://web.archive.org/web/20240606081039/https://sjtug.org/post/mirror-news/2024-06-06-takedown-dockerhub/)

[![上海交大通知](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/68747470733a2f2f62322e77776b656a697368652e746f702f57502d43444e2d30322f323032342f3230323430363038323134373037362e77656270)](https://camo.githubusercontent.com/b32ad4bc54fa9d0a008603116a19cb49aa83e0373fe98c75271547ba96b783ef/68747470733a2f2f62322e77776b656a697368652e746f702f57502d43444e2d30322f323032342f3230323430363038323134373037362e77656270)

**Dockerhub官网**

官方网站：https://hub.docker.com/



### 11.2 DockerHub国内镜像加速源列表

国内使用 Docker 的朋友们，可能都遇到过配置镜像源来加速镜像拉取的操作。然而，最近几个月发现许多曾经常用的国内镜像站（包括各种云服务商和高校镜像站）已经无法使用。

此列表只收录目前可用的 DockerHub 镜像站和镜像加速地址，感谢这些公益服务者。

> 请注意！有些镜像站仅提供基础镜像或白名单镜像，如果某个加速地址无法拉取到所需的镜像，可以尝试切换到其他地址。有些代理站点是热心网友自费搭建的，请务必合理使用。

**推荐镜像代理仓库**

| DockerHub镜像仓库                                            | 镜像加速器地址                            |
| ------------------------------------------------------------ | ----------------------------------------- |
| [本博客镜像站](https://docker.zycloud.tk)                    | `https://docker.zycloud.tk`               |
| [毫秒镜像](https://1ms.run/)                                 | `https://docker.1ms.run`                  |
| [DaoCloud 镜像站](https://github.com/DaoCloud/public-image-mirror) | `https://docker.m.daocloud.io`            |
| [腾讯云](https://cloud.tencent.com/document/product/457/9113)（只支持内网访问，不支持外网域名访问加速） | `https://mirror.ccs.tencentyun.com`       |
| [阿里云](https://cr.console.aliyun.com/)（需登录，系统分配） | `https://<your_code>.mirror.aliyuncs.com` |



### 11.3 配置Dockerhub镜像源使用教程

**方法一：修改配置文件**

为了加速镜像拉取，使用以下命令设置 **registry mirror**

> 支持系统：Ubuntu 16.04+、Debian 8+、CentOS 7+

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://docker.zycloud.tk",
    "https://docker.1ms.run",
    "https://docker.m.daocloud.io"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



**方法二：使用 DockerHub代理**

以[本博客镜像站](https://docker.zycloud.tk)为例：可以根据列表自行替换来测试是否拉取成功

```
docker pull docker.zycloud.tk/nginx:latest
```

说明：国内代理加速网速可能会受到一定限制



### 11.4 DockerHub containerd的配置文件

配置文件参考：

```
sudo tee /etc/containerd/config.toml <<EOF
[plugins."io.containerd.grpc.v1.cri".registry]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
    [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
      endpoint = [
        "https://docker.zycloud.tk",
        "https://docker.1ms.run",
        "https://docker.m.daocloud.io"
      ]
EOF
sudo systemctl daemon-reload
sudo systemctl restart containerd
```
