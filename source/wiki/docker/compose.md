---
layout: wiki  # 使用wiki布局模板
wiki: docker # 这是项目id，对应 /_data/wiki/docker.yml
title: 12. Docker Compose教程
order: 11
---

# 12. Docker Compose教程

本教程全面概述了用于管理容器的基本 Docker Compose 命令。无论你是 Docker 新手还是有经验的用户，本指南都将为你提供必要的知识，以便使用 Docker Compose 高效地编排和管理你的容器化应用程序。



## 12.1 开始使用 Docker Compose

Docker Compose 是一个强大的工具，可简化管理和部署多容器应用程序的过程。它允许你使用简单的 YAML 配置文件来定义和运行复杂的应用程序，从而更轻松地管理容器的生命周期。

### 了解 Docker Compose

Docker Compose 是 Docker 生态系统的一部分。它用于定义和运行多容器 Docker 应用程序。使用 Docker Compose，你可以创建一个单一的配置文件，该文件定义应用程序所需的所有服务、网络和卷，从而轻松地整体部署和管理应用程序。

### 安装 Docker Compose

要在 Ubuntu 22.04 系统上安装 Docker Compose，你可以按照以下步骤操作：

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose 
```

这将下载最新版本的 Docker Compose 并将其安装到你的系统上。

### 创建 Docker Compose 文件

Docker Compose 的核心是 YAML 配置文件，通常命名为 `docker-compose.yml`。此文件定义构成应用程序的服务、网络和卷。以下是一个示例：

```yaml
version: "3"
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: password
```

此配置文件定义了两个服务：运行 Nginx 的 Web 服务器和 MySQL 数据库。

### 运行 Docker Compose 应用程序

创建 `docker-compose.yml` 文件后，你可以使用以下命令启动应用程序：

```bash
docker-compose up -d AI讲解 立即练习
```

这将以分离模式启动配置文件中定义的所有服务，使你能够继续使用终端。

### 扩展和管理容器

Docker Compose 还使你能够通过增加或减少特定服务的副本数量来轻松扩展应用程序。你可以使用 `docker-compose scale` 命令来实现：

```bash
docker-compose scale web=3 AI
```

这将把 `web` 服务扩展到 3 个副本。

## 12.2 Docker Compose 命令

Docker Compose 提供了一系列用于管理容器的命令。以下是一些你应该了解的最重要的命令：

### 构建和启动容器

- `docker-compose build`：构建或重新构建服务。
- `docker-compose up`：启动所有服务。
- `docker-compose up -d`：以分离模式启动所有服务。

### 停止和移除容器

- `docker-compose stop`：停止正在运行的服务。
- `docker-compose down`：停止并移除容器、网络、镜像和卷。

### 列出和检查容器

- `docker-compose ps`：列出所有正在运行的容器。
- `docker-compose logs`：显示服务的日志输出。
- `docker-compose config`：验证并查看 Compose 文件。

### 扩展和更新容器

- `docker-compose scale`：向上或向下扩展服务。
- `docker-compose up --scale web=3`：将 `web` 服务扩展到 3 个副本。
- `docker-compose pull`：拉取服务的最新镜像。
- `docker-compose push`：推送服务镜像。

### 管理卷和网络

- `docker-compose volume ls`：列出所有卷。
- `docker-compose volume rm`：移除一个或多个卷。
- `docker-compose network ls`：列出所有网络。
- `docker-compose network rm`：移除一个或多个网络。

这些命令提供了一套全面的工具来管理基于 Docker Compose 的应用程序。通过理解和使用这些命令，你可以有效地管理容器的生命周期，并确保多容器应用程序的顺利运行。

## 12.3 Docker Compose 使用的实际示例

在本节中，我们将探讨一些如何使用 Docker Compose 来管理应用程序的实际示例。

### 示例 1：部署 Web 应用程序和数据库

假设我们有一个需要 MySQL 数据库的 Web 应用程序。我们可以创建一个 `docker-compose.yml` 文件来定义和管理此设置：

```yaml
version: "3"
services:
  web:
    image: myapp/web:latest
    ports:
      - "80:8080"
    depends_on:
      - db
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: password
    volumes:
      - db-data:/var/lib/mysql
volumes:
  db-data: AI
```

在此示例中，我们有两个服务：`web` 和 `db`。`web` 服务运行我们的 Web 应用程序，而 `db` 服务运行 MySQL 数据库。`depends_on` 字段确保 `web` 服务在 `db` 服务之后启动。

要部署此应用程序，我们可以运行以下命令：

```bash
docker-compose up -d AI
```

这将以分离模式启动应用程序，使你能够继续使用终端。

### 示例 2：扩展服务

假设我们想要扩展我们的 Web 应用程序以处理更多流量。我们可以使用 `docker-compose scale` 命令来实现：

```bash
docker-compose scale web=3
```

这将把 `web` 服务扩展到 3 个副本，使我们能够将负载分布到多个容器上。

### 示例 3：部署多层应用程序

Docker Compose 对于管理多层应用程序特别有用，例如具有前端、后端和数据库的 Web 应用程序。以下是一个示例：

```yaml
version: "3"
services:
  frontend:
    image: myapp/frontend:latest
    ports:
      - "80:80"
    depends_on:
      - backend
  backend:
    image: myapp/backend:latest
    environment:
      DB_HOST: db
    depends_on:
      - db
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: password
    volumes:
      - db-data:/var/lib/mysql
volumes:
  db-data: AI
```

在此示例中，我们有三个服务：`frontend`、`backend` 和 `db`。`frontend` 服务依赖于 `backend` 服务，而 `backend` 服务依赖于 `db` 服务。这确保应用程序按正确顺序部署。

通过使用 Docker Compose，你可以轻松管理多层应用程序的部署和扩展，使其成为基于容器的开发和部署的强大工具。

## 12.4 总结

在本教程中，你已经学习了用于容器管理的关键 Docker Compose 命令，从开始使用 Docker Compose 到探索其使用的实际示例。通过掌握这些基本的 Docker Compose 命令，你可以简化容器编排流程，并确保应用程序的顺利部署和管理。
