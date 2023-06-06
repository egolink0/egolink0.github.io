---
title: docker 之使用 compose 文件启动容器 clickhouse-server
date: 2023-06-05 20:00:00 +0800
categories: [文章, docker]
tags: [docker,docker-compose,clickhouse]
author: ego
---

## docker-compose 安装

```shell
# linux 安装 docker-compose
curl -L https://get.daocloud.io/docker/compose/releases/download/v2.4.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 修改执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 创建软连接
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

## clickhouse 对应的 docker-compose.yaml
- 指定配置文件起服务：  
`docker-compose -f ./xxx.yaml up -d`   
- 不指定文件名称，会在当前文件夹下找 `docker-compose.yaml` 文件，进行编译   
`docker-compose up -d`  

```yaml
# ClickHouse Document: https://clickhouse.com/docs/en/
# ClickHouse Docker Hub: https://hub.docker.com/r/yandex/clickhouse-server
services:
  clickhouse: # 服务名称，可以多个
    image: yandex/clickhouse-server:latest # 使用的镜像名称，可以是本地地址
    container_name: clickhouse-server # 容器名称
    restart: always
    ulimits:
      nofile:
        soft: 262144
        hard: 262144
    ports: # 端口映射
      - "8123:8123"
      - "9000:9000"
      - "9009:9009"
    volumes: # 数据映射
      - /data/clickhouse/data:/var/lib/clickhouse:rw 
      - /data/clickhouse/log:/var/log/clickhouse-server:rw
      - /data/clickhouse/config/config.xml:/etc/clickhouse-server/config.xml
      - /data/clickhouse/config/users.xml:/etc/clickhouse-server/users.xml

```