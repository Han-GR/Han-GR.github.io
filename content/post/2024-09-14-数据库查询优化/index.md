---
title: "数据库查询优化"
description: ""
slug: 2024-09-14-数据库查询优化
date: 2024-09-14
image: ""
categories:
  - "数据库&缓存"
tags:
  - "数据库"
  - "数据库查询优化"
  - "缓存"
---
这篇记录一次“造大量数据做压测”引出的数据库性能问题：先把数据插进去，再聊怎么让查询不慢。

## 背景

- 为了测试查询，向数据库插入了大量数据（约 1000 万条记录）
- 数据插入完成后，查询变慢，需要给出优化思路

## 数据库插入数据的几种方式

### 1) 可视化工具

适合一次性插入少量数据、数据量可控时使用：

- IDEA DataGrip
- Navicat

### 2) SQL 语句

适合数据量较小时使用：

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

### 3) 程序执行

适合大数据量时使用：

- for 循环生成数据，建议分批插入
- 需要保证可控、幂等，并注意线上环境与测试环境差异
- 常见选择：Python / Java

## Java 示例：两种插入方式

### 1) 单线程插入

分批次插入，速度较慢：

```java
@SpringBootTest
public class InsertUsersTest {

    @Resource
    private UserService userService;

    /**
     * 批量插入用户信息
     */
    @Test
    public void doInsertUsers() {
        StopWatch stopWatch = new StopWatch("InsertUsers");
        stopWatch.start();
        final int count = 100000;

        List<User> userList = new ArrayList<>();

        for (int i = 0; i < count; i++) {
            User user = new User();
            user.setUserName("xxx");
            user.setUserAccount("xxxx");
            user.setUserAvatar("xxxx");
            user.setGender(0);
            user.setUserPassword("12345678");
            user.setPhone("13812345678");
            user.setEmail("234234234@qq.com");
            user.setUserRole("user");
            user.setTags("[]");
            user.setUserProfile("我们都是假数据, 请勿相信！");
            userList.add(user);
        }
        userService.saveBatch(userList, 100);
        stopWatch.stop();
        System.out.println(stopWatch.getTotalTimeMillis());
        stopWatch.getTotalTimeMillis();
    }
}
```

### 2) 多线程插入

利用多线程并发插入，速度较快：

```java
@SpringBootTest
public class InsertUsersTest {

    @Resource
    private UserService userService;

    /**
     * 线程池
     * ThreadPoolExecutor 是 Java 中用于创建和管理线程池的类, 它的构造函数接受几个参数, 用于配置线程池的行为. 具体参数的含义如下: 
     * corePoolSize (60): 这是线程池中始终保持在活动状态的线程数量. 当新的任务到来时, 如果当前的线程数小于这个核心数量, 即使其他线程处于闲置状态, 线程池也会创建新的线程来处理任务. 
     * maximumPoolSize (1000): 这是线程池允许的最大线程数. 当工作线程的数量达到核心池大小时, 如果有新的任务到来且队列未满, 线程池会创建新的线程, 直到达到这个最大值. 
     * eepAliveTime (10000): 这个参数指定了多余的空闲线程在终止之前会保持多久的时间. 在这个时间段内, 如果线程没有任务可执行, 它会被终止. 此处使用 TimeUnit.MINUTES 表示时间单位是分钟. 
     * unit (TimeUnit.MINUTES): 这个参数指定了 keepAliveTime 的时间单位. 
     * workQueue (new ArrayBlockingQueue<>(1000)): 这是一个阻塞队列, 用于存储等待执行的任务. 当所有核心线程都在忙于执行任务时, 新提交的任务会被放入这个队列中. ArrayBlockingQueue 是一个基于数组的阻塞队列, 容量设置为 1000. 
     * 这个线程池配置了一个核心线程数为 60, 最大线程数为 1000, 空闲线程的保活时间为 10000 分钟（相对较长）, 并且使用一个容量为 1000 的阻塞队列来存储等待执行的任务. 这样的配置可以有效地处理高并发插入操作. 
     */
    private ExecutorService executorService = new ThreadPoolExecutor(60, 1000, 10000, TimeUnit.MINUTES, new ArrayBlockingQueue<>(1000));

    /**
     * 并发插入用户信息
     */
    @Test
    public void doInsertUsersConcurrently() {
    }
}
```
