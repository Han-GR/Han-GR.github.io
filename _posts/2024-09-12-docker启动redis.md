---
layout: mypost
title: docker启动redis
categories: [ docker, redis ]
---

### 前言

- 有项目使用redis, 本来想在阿里云直接购买云redis, 但是发现阿里云价格不便宜, 所以决定使用docker部署redis.

<br>
<br>

### 步骤

<br>

#### 1. 安装docker

- 可以查看之前的文章[docker安装](https://han-gr.github.io/posts/2024/08/31/docker%E5%AE%89%E8%A3%85%E9%97%AE%E9%A2%98.html)

<br>

#### 2. 创建Dockerfile文件

```dockerfile
# 使用官方的Redis镜像作为基础镜像
FROM redis:latest

# 设置Redis配置文件
COPY redis.conf /usr/local/etc/redis/redis.conf

# 暴露Redis端口
EXPOSE 6379

# 启动Redis并应用配置文件
CMD ["sh", "-c", "redis-server /usr/local/etc/redis/redis.conf"]
```

<br>

#### 3. 创建redis.conf文件

```conf
# redis.conf
bind 0.0.0.0        # 允许远程连接
port 6379           # 端口号
timeout 0           # 客户端连接超时时间
loglevel notice         # 日志级别
logfile ""            # 日志文件名
databases 16         # 数据库数量
save 900 1        # 自动保存时间间隔
save 300 10       # 自动保存时间间隔
save 60 10000      # 自动保存时间间隔
rdbcompression yes       # 是否压缩rdb文件
dbfilename dump.rdb         # rdb文件名
dir /data        # 数据文件存放目录
requirepass xxxxxx       # 密码
maxmemory 2gb      # 最大内存
maxmemory-policy noeviction     # 内存淘汰策略
```

<br>

#### 4. 构建镜像

```bash
docker build -t redis:latest .
```

<br>

#### 5. 运行容器

```bash
docker run -dit -p 6379:6379 --name redis redis:latest
```

