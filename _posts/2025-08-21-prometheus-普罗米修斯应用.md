---
layout: mypost
title: prometheus-普罗米修斯学习应用
categories: [ prometheus, 普罗米修斯 ]
---

<br>

## 背景

- 网站 [Picazza](https://picazza.cn/) 上线之后, 虽然后台有记录日志, 但一直想弄个监控系统来监控网站的运行状态,
  便学习了prometheus.

<br>

## prometheus概述

### 介绍

- Prometheus是一个开源的系统监控和报警工具,它最初是SoundCloud公司内部使用的监控系统.
- 是一款时序性数据库,可以存储和查询时序数据,支持PromQL（Prometheus Query Language）查询语言.

<br>

### 特性

#### 1. 多维数据模型

- Prometheus 使用多维数据模型,通过指标名称和键值对（标签）来标识数据.这种模型使得用户可以灵活地对数据进行聚合和过滤,从而进行详细的分析.

#### 2. PromQL 查询语言

- Prometheus 提供了一种名为 PromQL（Prometheus Query Language）的强大查询语言,用户可以用它来进行实时的数据查询和分析.这种查询语言设计直观,功能强大,适合复杂的数据操作和聚合.

#### 3. 时间序列数据库

- Prometheus 内置了一个高效的时间序列数据库,用于存储和检索监控数据.数据以时间序列的形式存储,每个时间序列由唯一的指标名和一组标签确定.

#### 4. 数据抓取模型

- Prometheus 采用 pull 模型,通过 HTTP 协议定期从被监控的服务抓取数据.这种方式使得 Prometheus
  可以很好地适应动态和分布式的环境,特别适用于微服务架构.

#### 5. 丰富的生态系统

- Prometheus 有丰富的生态系统,支持多种导出器（Exporter）,可以与许多不同的服务和应用集成.例如：
    - Node Exporter：用于监控 Linux 系统的基本资源指标.
    - Blackbox Exporter：用于探测网络服务的可用性.
    - Custom Exporter：用户可以编写自定义导出器来监控特定的应用和服务.

#### 6. 报警功能

- Prometheus 内置了报警功能,用户可以根据设定的规则生成报警.报警规则使用 PromQL 定义,并可以通过 Alertmanager
  发送通知,支持多种通知方式（如电子邮件、Slack、PagerDuty 等）.

#### 7. 服务发现

- Prometheus 支持多种服务发现机制,可以自动发现和监控动态变化的服务.这对于 Kubernetes 等容器编排系统特别有用.

#### 8. 可视化工具

- Prometheus 通常与 Grafana 一起使用.Grafana 是一个开源的可视化工具,提供了强大的数据展示和仪表盘功能,用户可以创建和分享丰富的监控仪表盘.

<br>

### 主要应用场景

- 云原生应用：适用于 Kubernetes 等容器化环境的监控.
- 微服务架构：监控复杂的微服务应用.
- 基础设施监控：监控服务器、网络设备和其他基础设施组件.

<br>

### 生态系统组件

- Prometheus Server：负责抓取和存储时间序列数据.
- Alertmanager：处理报警通知.
- Pushgateway：用于短期作业的指标推送.
- Prometheus Exporters：用于导出指标数据的工具.
- Prometheus 以其灵活性、高性能和广泛的社区支持,成为现代监控系统的首选之一.

<br>

## prometheus架构

- ![prometheus架构图](img.png)

### 组件介绍

#### 1. Prometheus Server

- Retrieval: Prometheus 服务器从各个目标（targets）抓取监控数据.目标可以是各种服务、应用和设备,通常通过 HTTP 协议抓取指标数据.
- TSDB (Time Series Database): 抓取到的数据存储在时间序列数据库中,用于后续的查询和分析.
- HTTP Server: 提供一个 HTTP 端点,用户可以通过它查询监控数据、查看仪表盘和管理配置.

#### 2. Service Discovery

- Prometheus 支持多种服务发现机制,如 Kubernetes、Consul、DNS 等,用于自动发现和监控动态变化的目标.
- kubernetes 和 file_sd 是两种常见的服务发现方式,分别用于从 Kubernetes 集群和文件中发现监控目标.

#### 3.Jobs/Exporters

- Jobs: 定义了要监控的一组服务或应用,每个 job 包含多个目标（targets）.
- Exporters: 特殊的服务,用于从各种系统和服务中导出监控指标.例如,Node Exporter 用于导出主机的系统级指标.

#### 4. Pushgateway

- 用于处理短期任务（short-lived jobs）的指标.这些任务可能在 Prometheus 抓取周期内结束,因此无法直接被 Prometheus
  抓取.Pushgateway 允许这些任务在退出时将指标推送到网关,Prometheus 再从 Pushgateway 中抓取这些数据.

#### 5. Alertmanager

- 处理由 Prometheus 服务器生成的报警（alerts）,根据配置的规则将报警通知发送到不同的接收渠道,如电子邮件、Slack、PagerDuty 等.

#### 6. Visualization and API Clients

- Prometheus Web UI: 提供了一个简单的界面,可以直接查询和查看监控数据.
- Grafana: 一个强大的开源数据可视化和监控工具,通常与 Prometheus 一起使用.Grafana 可以创建复杂的仪表盘来展示监控数据.
- API Clients: 提供各种 API,用于与其他系统和应用集成.

<br>

### 工作流程总结

- 数据抓取: Prometheus 服务器通过服务发现或静态配置,定期从各个目标（targets）抓取监控数据.
- 数据存储: 抓取的数据存储在时间序列数据库（TSDB）中.
- 报警生成: 根据配置的规则,Prometheus 服务器会生成报警,并将这些报警推送到 Alertmanager.
- 报警通知: Alertmanager 根据配置的通知渠道,将报警通知发送给相关人员.
- 数据查询和可视化: 用户可以通过 Prometheus Web UI 或 Grafana 查询和可视化监控数据.

通过这种架构设计,Prometheus 提供了一个灵活、高效且可扩展的监控和报警解决方案,适用于现代云原生和分布式系统的监控需求.

<br>

## prometheus安装

### 准备镜像

- 国内现在没有办法直接拉取镜像了, 所以我把镜像上传到阿里云的容器镜像管理中
-
我在之前的文章里介绍过,详情参考 [docker安装elasticsearch和kibana](https://han-gr.github.io/posts/2024/10/08/docker%E5%AE%89%E8%A3%85elasticsearch%E5%92%8Ckibana.html)

### 编写配置文件

- prometheus.yml

```
# Prometheus 配置文件
global:
  scrape_interval: 15s      # 全局抓取间隔
  evaluation_interval: 15s  # 规则评估间隔


# 抓取配置
scrape_configs:
  # Prometheus 自身监控
  - job_name: 'prometheus'
    metrics_path: /metrics
    static_configs:
      - targets: ['localhost:9090']
      
 
  # Spring Boot 应用监控
  - job_name: 'XXXXX'
    metrics_path: '/api/actuator/prometheus'  # Spring Boot Actuator 端点
    static_configs:
      - targets: ['localhost:8080']  # 应用服务器地址
```

### 准备dockerfile

```
# 使用阿里云私有仓库的镜像作为基础镜像
FROM registry.cn-beijing.aliyuncs.com/xxxxxx/prometheus:main

# 设置prometheus配置文件
COPY prometheus.yml /etc/prometheus/prometheus.yml
```

### 构建镜像

```
docker build -t prometheus .
```

### 启动容器

- Prometheus 数据存储在容器内的 /prometheus 目录中,因此每次容器重启时数据都会被清除.为了保存数据,需要为容器设置持久存储（或绑定挂载）.

```
# 创建数据卷
docker volume create prometheus-data

# 启动容器
docker run -dit --name prometheus -p 9090:9090 -v prometheus-data:/prometheus prometheus:latest
```

### 验证

- 访问 http://localhost:9090(或容器所在服务器的IP地址) 即可看到 Prometheus 的监控页面.
- 输入查询语句,点击 Execute 按钮即可查询监控数据.
- ![界面](img_1.png)

<br>

## 参考

- [Prometheus 官网](https://prometheus.io/)
- [Prometheus 官方文档](https://prometheus.io/docs/introduction/overview/)
- [运维锅总详解Prometheus](https://cloud.tencent.com/developer/article/2433939)


