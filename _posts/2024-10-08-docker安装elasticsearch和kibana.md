---
layout: mypost
title: docker安装elasticsearch和kibana
categories: [ docker, elasticsearch, kibana ]
---

<br>

### 背景

- 在项目学习中使用到了Elasticsearch, 因此需要安装Elasticsearch和Kibana.
- 本地windows也可以安装, 但是我把所有的环境都放到docker中了, 所以自己摸索着在docker中安装Elasticsearch和Kibana.

<br>

### 准备镜像

- 我使用的Elasticsearch和Kibana的版本都是7.17.23, 因此需要拉取对应的镜像.
- 但是国内现在没有办法直接拉取镜像了, 所以我通过一些方法把镜像上传到阿里云的容器镜像管理中
- 使用的是开源项目: [https://github.com/tech-shrimp/docker_image_pusher](https://github.com/tech-shrimp/docker_image_pusher)
- 按照该项目的说明将Elasticsearch和Kibana的镜像上传到阿里云镜像仓库, 也方便以后使用.
- ![阿里云镜像仓库](img.png)

<br>

### 准备网络

- Elasticsearch和Kibana需要在同一个docker网络中才能通信, 因此创建一个docker网络.
- 运行命令: `docker network create es-net`

<br>

### 安装Elasticsearch

1. 拉取镜像
    - 使用阿里云镜像仓库的地址拉取镜像, 镜像仓库中有操作指南可参考
    - ![拉取镜像操作指南](img_1.png)
2. 启动容器
    - 先在宿主机上创建一个目录作为数据存储
       ```bash
       mkdir -p es/{config,data,logs,plugins}
       ```
        - ![创建数据目录](img_2.png)
        - **注意**: data, logs, plugins目录需要给权限, 否则容器启动会报错.
    - 然后在config目录中创建elasticsearch.yml文件, 内容如下:
       ```yml
       cluster.name: "docker-cluster"  # 设置集群名称
       network.host: 0.0.0.0  # 允许外部访问
       http.cors.enabled: true  # 允许跨域请求
       http.cors.allow-origin: "*"  # 允许跨域请求的源
       http.cors.allow-headers: Authorization  # 允许跨域请求的header
       xpack.security.enabled: true  # 开启xpack安全插件
       xpack.security.transport.ssl.enabled: true  # 开启xpack安全插件的SSL
       ```
    - 最后使用命令启动容器(为了解释方便, 我把命令分开写, 其实是一条命令, 中间用空格隔开)
       ```bash
       docker run -dit   # 后台运行容器
       --name es   # 容器名称
       --net es-net   # 指定网络
       -m 1GB -e"ES_JAVA_OPTS=-Xms512m -Xmx1024m"  # 设置内存和JVM参数 
       -e "discovery.type=single-node"   # 单节点集群
       -v ./softTools/es/data:/usr/share/elasticsearch/data     # 挂载数据目录
       -v ./softTools/es/plugins:/usr/share/elasticsearch/plugins  # 挂载插件目录
       -v ./softTools/es/logs:/usr/share/elasticsearch/logs  # 挂载日志目录
       -v ./softTools/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml     # 挂载配置文件
       --privileged   # 开启特权模式
       -p 9200:9200   # 端口映射
       elasticsearch # 镜像名称
       ```
3. 验证安装成功
    - 浏览器访问: http://ip:9200
    - 出现如下页面则安装成功
    - ![验证安装成功](img_3.png)

<br>

### 安装Kibana

1. 拉取镜像
    - 参考Elasticsearch安装步骤
    - 使用阿里云镜像仓库的地址拉取镜像, 镜像仓库中有操作指南可参考
2. 启动容器
    - 先在宿主机上创建一个目录, 作为配置文件目录
       ```bash
       mkdir -p kibana/config
       ```
    - 然后在config目录中创建kibana.yml文件, 内容如下:
       ```yml
       #
       # ** THIS IS AN AUTO-GENERATED FILE **
       #
 
       # Default Kibana configuration for docker target
       server.host: "0.0.0.0"  # 允许外部访问
       server.shutdownTimeout: "5s"  # 设置关闭超时时间
       elasticsearch.hosts: [ "http://elasticsearch:9200" ]  # 设置Elasticsearch地址
       monitoring.ui.container.elasticsearch.enabled: true  # 开启监控
       i18n.locale: "zh-CN"  # 设置语言为中文
       elasticsearch.username: "xxxxxx"  # 设置用户名
       elasticsearch.password: "xxxxxxx"  # 设置密码
       ```
    - 最后使用命令启动容器(为了解释方便, 我把命令分开写, 其实是一条命令, 中间用空格隔开)
       ```bash
       docker run -dit   # 后台运行容器
       --name kibana   # 容器名称
       -v ./softTools/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml   # 挂载配置文件
       -e ELASTICSEARCH_HOSTS=http://es:9200   # 设置Elasticsearch地址, 注意容器名称, 我的Elasticsearch容器名称为es
       --net=es-net   # 指定网络
       -p 5601:5601   # 端口映射
       kibana   # 镜像名称
       ```
3. 验证安装成功
    - 浏览器访问: http://ip:5601
    - 出现如下页面则安装成功, 输入用户名和密码登录即可
    - ![验证安装成功](img_4.png)
    - ![登录成功](img_5.png)

<br>

### 安装分词插件

1. 下载分词插件(打开网址找你需要的版本)
    - 中文分词插件: [https://release.infinilabs.com/analysis-ik/stable/](https://release.infinilabs.com/analysis-ik/stable/)
    - 拼音分词插件: [https://release.infinilabs.com/analysis-pinyin/stable/](https://release.infinilabs.com/analysis-pinyin/stable/)
2. 解压插件到Elasticsearch的plugins目录下
    ```bash
    mkdir -p es/plugins/{ik,pinyin}
    cd es/plugins/ik
    wget https://release.infinilabs.com/analysis-ik/stable/elasticsearch-analysis-ik-7.17.23.zip
    unzip elasticsearch-analysis-ik-7.17.23.zip
    cd ../pinyin
    wget https://release.infinilabs.com/analysis-pinyin/stable/elasticsearch-analysis-pinyin-7.17.23.zip
    unzip elasticsearch-analysis-pinyin-7.17.23.zip
    ```
    - **注意**: 解压插件之后要将压缩包删除, 否则Elasticsearch启动会报错.
3. 重启Elasticsearch容器
    ```bash
    docker restart es
    ```

