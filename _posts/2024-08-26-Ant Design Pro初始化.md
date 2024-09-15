---
layout: mypost
title: Ant Design Pro 初始化过程
categories: [ Ant Design Pro, 前端, 用户中心项目 ]
---

### 执行安装命令

- 参考 [Ant Design Pro官网](https://pro.ant.design/zh-CN/docs/getting-started)

<br>

#### 1.初始化

```bash
# 使用 npm
npm i @ant-design/pro-cli -g
pro create myapp
```

<br>

#### 2.选择 umi 的版本

- 如果选择了 umi@4 版本, 暂时还不支持全量区块. 
- 如果选择了 umi@3, 还可以选择 pro 的模板, pro 是基础模板, 只提供了框架运行的基本内容, complete 包含所有区块,
  不太适合当基础模板来进行二次开发

```
? 🐂 使用 umi@4 还是 umi@3 ? (Use arrow keys)
❯ umi@4
  umi@3
```

```
? 🚀 要全量的还是一个简单的脚手架? (Use arrow keys)
❯ simple
  complete
```

<br>

#### 3.安装依赖: 

```bash
$ cd myapp && tyarn
// 或
$ cd myapp && npm install
```

<br>

#### 4.启动

- 打开项目, 在 package.json 文件中找到 start 配置, 点击运行
- ![运行](img.png)

<br>

#### 5.查看效果

- 打开 [本地浏览器请求地址](http://localhost:8000/) 初始化完成
- ![浏览器运行效果](img.png)
