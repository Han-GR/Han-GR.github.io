---
title: "Docker 安装问题"
description: ""
slug: 2024-08-31-Docker-安装问题
date: 2024-08-31
image: ""
categories:
  - "运维&云原生"
tags:
  - "Docker"
  - "Linux"
---

### 背景

- 需要使用 docker 部署项目, 服务器环境是 CentOS 7.9, 安装 docker 遇到了一些问题


### 问题

1. 描述
    - 使用 yum 命令安装了 docker 并且安装过程没有出现错误, 但是显示找不到服务

2. 原因
    - 经过查看了解发现, CentOS 默认使用 podman 代替 docker, 所以需要将 podman 卸载


### 解决流程

1. 卸载 podaman

    ```bash
    yum erase podman buildah
    ```

2. 安装依赖环境

    ```bash
    yum install -y yum-utils
    ```

3. 安装配置镜像

    ```bash
    sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    ```

4. 安装 docker CE

    ```bash
    sudo yum install -y docker-ce docker-ce-cli containerd.io
    ```

5. 启动 Docker 服务

    ```bash
    systemctl start docker
    ```