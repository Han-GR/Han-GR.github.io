---
layout: mypost
title: 后端 Dockerfile 模板
categories: [Java, maven, Docker, 后端]
---

```
# 自己制作的 jdk21 + mvn3.9.9 的 Docker 镜像
FROM registry.cn-beijing.aliyuncs.com/hengjixiang/base-image:v2

# 为镜像设置作者标签,可不写
LABEL authors="Han"

# 设置工作目录为/app
WORKDIR /app

# 将当前目录下的pom.xml文件复制到工作目录/app中
COPY pom.xml .

# 将当前目录下的src文件夹复制到工作目录/app中的src文件夹
COPY src ./src

# 在工作目录/app中运行Maven命令，打包项目并跳过测试
RUN mvn package -DskipTests

# 设置容器启动时执行的命令，使用java命令运行打包好的jar文件，并在prod(生产环境)运行
CMD ["java", "-jar", "/app/target/uc-back-0.0.1-SNAPSHOT.jar",  "--spring.profiles.active=prod"]
```