---
title: docker 之部署 clickhouse-server 及数据存储（二）
date: 2023-06-04 20:00:00 +0800
categories: [文章, docker]
tags: [docker,clickhouse]
author: ego
---

## 1. 在本机创建同步数据目录
数据存储在 `/data/clickhouse` 目录下  
```shell
# 创建基本目录结构
mkdir -p /data/clickhouse/data /data/clickhouse/config /data/clickhouse/log  

# 查看创建的目录
tree /data 
# data
# └── clickhouse
#     ├── config # 存基本配置文件
#     ├── data # 映射数据文件
#     └── log # 映射日志
```

## 2. 拷贝 clickhouse 配置文件
### 2.1 启服务
```shell
docker run -d \
--name clickhouse-server \  # --name 命名容器
--ulimit nofile=262144:262144 \
-p 8123:8123 \
-p 9000:9000 \
-p 9009:9009 \
yandex/clickhouse-server:latest
```

### 2.2 将容器内部的配置文件拷贝到宿主机  
主要是 `config.xml` 和 `users.xml` 拷贝到宿主机目录 `/data/clickhouse/config` 下 
```shell
# config.xml
docker cp clickhouse-server:/etc/clickhouse-server/config.xml /data/clickhouse/config/config.xml

# users.xml
docker cp clickhouse-server:/etc/clickhouse-server/users.xml /data/clickhouse/config/users.xml
```

## 3. 修改 clickhouse 密码
### 3.1 生成 sha256 密码
```shell
PASSWORD=$(base64 < /dev/urandom | head -c14); echo "123456"; echo -n "123456" | sha256sum | tr -d '-'
# 输出如下
# 123456 # 明文密码
# 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92 # 加密后密码
```

### 3.2 修改 users.xml 配置文件里面的密码
```shell 
vim /data/clickhouse/config/users.xml
# 替换 <password></password>  为<password_sha256_hex>8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92</password_sha256_hex>  # 中间是密码 123456 加密的密文
```

### 3.3 增加一个 root 用户
找到 users 里面的 default 用户，在下面增加 root 用户的配置
```xml
<users>
    <default>xxx</default>  <!--default 用户的配置-->
    <root>xxx</root> <!--新增的 root 用户配置-->
</users>
```

root 用户配置
```xml
<root>   <!--用户名：root-->
<!--     <password>root</password>  明文密码，官网不推荐使用明文 -->
    <password_sha256_hex>4813494d137e1631bba301d5acab6e7bb7aa74ce1185d456565ef51d7      37677b2</password_sha256_hex> <!--加密密码：sha256-->
    <networks incl="networks" replace="replace">  <!--网络设置，限制可登陆的客户端地址-->
      <ip>::/0</ip>  <!--为所有客户端打开权限-->
    </networks>
    <profile>default</profile>
    <quota>default</quota>
</root>
```

## 4 部署 clickhouse-server
### 4.1 先移除刚才创建的容器
```shell
docker rm -f 容器ID
```

### 4.2 部署 clickhouse-server 并挂载数据目录
```shell 
docker run -d \
--name clickhouse-server \
--ulimit nofile=262144:262144 \
-p 8123:8123 \
-p 9000:9000 \
-p 9009:9009 \
# -v 挂载目录，同步宿主目录到容器目录
-v /data/clickhouse/data:/var/lib/clickhouse:rw \  
# :rw 执行读写两个权限 r:read,w:write linux 文件权限
-v /data/clickhouse/log:/var/log/clickhouse-server:rw \  
-v /data/clickhouse/config/config.xml:/etc/clickhouse-server/config.xml \
-v /data/clickhouse/config/users.xml:/etc/clickhouse-server/users.xml \
yandex/clickhouse-server:latest
```
现在就可以用 root 123456 登录现在的容器 clickhouse-server 了。  
即使容器被销毁，下次重新用上面的部署命令启一个容器也会同步最新数据。  

### 4.3 最终 `/data` 目录长这样 
```text
/data
└── clickhouse
    ├── config
    │   ├── config.xml
    │   └── users.xml
    ├── data
    │   ├── access [error opening dir]
    │   ├── data [error opening dir]
    │   ├── dictionaries_lib [error opening dir]
    │   ├── flags [error opening dir]
    │   ├── ...
    └── log
        ├── clickhouse-server.err.log
        └── clickhouse-server.log
```


## 参考
[clickhouse 新建用户和密码](https://www.cnblogs.com/yoyo1216/p/12835941.html)