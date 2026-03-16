---
title: "knife4j不能上传文件的解决方案"
description: ""
slug: 2024-12-17-knife4j不能上传文件的解决方案
date: 2024-12-17
image: ""
categories:
  - "后端"
tags:
  - "knife4j"
  - "springBoot"
---

### 问题描述

- 使用knife4j工具，在线调试接口时，上传文件功能无法使用
- ![image](assets/file-20260310210238606.png)

### 解决方案

- 在 controller 方法中，将上传文件参数加上, @RequestPart(“file”)MultipartFile file注解即可

```java
public R<String> addFile(@RequestPart("file") MultipartFile file){
    // 处理上传文件逻辑
    return file;
} 
```

- ![image](assets/file-20260310210251313.png)

