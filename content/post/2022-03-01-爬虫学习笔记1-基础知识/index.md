---
title: 爬虫学习笔记1-基础知识
description: ""
slug: 2022-03-01-爬虫学习笔记1-基础知识
date: 2022-03-01
image: crawlSpider.jpg
categories:
  - AI&数据
tags:
  - python
  - 爬虫
---
工作也半年了，基本都在部署脚本，也一直在学习，今年的主要任务是爬虫和后端接口，后端主要使用flask，也在学习中，先学习爬虫


### 基本定义：

爬虫本质是模拟浏览器行为，自动化获取网页数据，但需遵守 robots.txt 和法律。

### 核心概念：

爬虫涉及网络协议、页面解析和数据存储。以下表格对比关键概念：

| 概念             | 定义与作用                  | 关键工具/技术                     | 常见问题               | 操作建议                       |
| -------------- | ---------------------- | --------------------------- | ------------------ | -------------------------- |
| **HTTP 协议**    | 网页传输协议，支持 GET/POST 等方法 | requests 库模拟请求              | 状态码 403/429（禁访/限速） | 用 `curl` 命令测试 URL          |
| **HTML/CSS**   | 页面结构/样式，选择器提取元素        | BeautifulSoup/lxml 解析       | 动态 JS 加载内容         | Chrome DevTools 检查元素       |
| **XPath**      | XML 路径语言，精确定位节点        | lxml.etree.XPath            | 语法复杂               | 用 `//div[@class='item']` 选 |
| **JSON/API**   | 结构化数据接口，直接返回数据         | requests.json() 解析          | API 密钥/限额          | 检查 Network 面板抓包            |
| **Robots.txt** | 网站爬虫协议，定义允许路径          | robotparser 库检查             | 忽略导致封禁             | 访问 `/robots.txt` 查看        |
| **User-Agent** | 浏览器标识，伪装请求头            | headers={‘User-Agent’: ‘…’} | 默认 Python 被识别      | 仿 Chrome UA 避检测            |

**解读**：爬虫 = 请求 + 解析 + 存储。基础版用 requests + BS4，高级用 Scrapy 框架。

### 主要库与工具：

Python 爬虫库多样，按复杂度分层：

#### 1\. **入门级：requests + BeautifulSoup**

-   **requests**：HTTP 请求神器，支持 GET/POST/Session。
-   **BeautifulSoup (BS4)**：HTML 解析器，友好 API。
-   **安装**：`pip install requests beautifulsoup4 lxml`（lxml 加速 BS4）。

#### 2\. **中级：Scrapy**

-   **Scrapy**：全栈框架，支持管道、爬虫引擎、中间件。
-   **安装**：`pip install scrapy`。

#### 3\. **高级：Selenium/Playwright**

-   **Selenium**：浏览器自动化，处理 JS 渲染。
-   **安装**：`pip install selenium webdriver-manager`。

#### 4\. **辅助库**

-   **fake-useragent**：随机 UA。
-   **proxies**：代理池（如 Scrapy-Redis）。
-   **Pandas**：数据存储/分析。
