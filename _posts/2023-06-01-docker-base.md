---
title: docker 之基础概念和使用（一）
date: 2023-06-01 20:00:00 +0800
categories: [文章, docker]
tags: [docker]
author: ego
---

# 主要概念
- **镜像**：Image，是用于创建 Docker 容器的模板，比如 Ubuntu 系统，类似本地的 一个个 git 仓库源码。
- **容器**：Container，是镜像运行时的实体，镜像和容器的关系就比如：**类（镜像）和实例（容器）**
- **仓库**：Repository，仓库可看成一个代码控制中心，用来保存镜像，类似本地的 git 仓库。
- **Docker Machine**: 是一个简化Docker安装的命令行工具
- **Dockerfile**: 是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明

- **Docker仓库**: 用来保存镜像，可以理解为代码控制中的代码仓库，类似 github。
 [Docker Hub](https://hub.docker.com) 提供了庞大的镜像集合供使用。


# 一次完整的操作
```
# 查看本地所有镜像
docker images

# 拉取最新的 ubuntu 镜像
docker pull ubuntu:latest

# 启动一个 ubuntu 的容器
sudo docker run -it -d -p 7000:7000 ubuntu:latest /bin/bash
# -p 指定映射端口：外部 8000 ，docker 容器内部 7000
# -d 后台方式运行
# -it 表示开启交互模式，并将用户当前的 shell 连接到容器内部，将 shell切换到容器的终端
# /bin/bash 运行 ubuntu 容器环境内部的此命令

# 将 shell 连接到正在后台运行的容器终端
sudo docker exec -it <容器ID> /bin/bash

# 退出 ubuntu 容器内部的终端，切换到用户 shell
exit 或者 Ctrl + D

# 确认正在运行的容器有哪些
docker ps
或者 docker container ls
docker ps -a # 查看所有容器，包括已经停止的容器 

# 启动/重启/停止/强制停止/移除 容器
docker container start/restart/stop/kill/rm <容器ID>
```

# 其他操作
```
# 查看容器日志
docker logs <容器ID>

# 查看容器资源消耗
docker stats 
```