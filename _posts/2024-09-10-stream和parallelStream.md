---
layout: mypost
title: stream和parallelStream
categories: [ java, stream, parallelStream, 踩坑 ]
---

<br>

### stream()和parallelStream()

- Java 8引入了Stream API, 它提供了一种高效且易于使用的处理数据的方式. Stream
  API可以用来对集合、数组、I/O流、生成器等进行操作, 并提供多种高级操作, 如filter()、map()、reduce()、sorted()等.
- Stream API的核心是Stream, 它是一个抽象概念, 它表示的是一个有序的、元素集合. Stream的操作分为中间操作和终止操作.
  中间操作返回一个Stream, 而终止操作则返回一个非Stream的结果.
- parallelStream()方法是stream()方法的并行版本, 它可以并行处理数据, 提高处理速度.

<br>

### Stream()的特点

- Stream()只能操作一次, 只能遍历一次.
- Stream()只能操作集合中的元素, 不能添加元素.
- Stream()操作之后, 原集合不会被修改.
- Stream()只能被消费一次, 消费完之后就没有内容可消费了.
- Stream()只能在集合元素类型相同的情况下使用.
- Stream()的操作是延迟执行的, 只有调用终止操作, 才会真正执行操作.
- Stream()的操作可以串行或并行执行.

<br>

### parallelStream()方法的特点

- parallelStream()方法返回的是并行流, 它可以在多个线程中执行, 它可以处理整个集合的数据.
- parallelStream()方法只能在集合元素类型相同的情况下使用.
- parallelStream()方法的操作是延迟执行的, 只有调用终止操作, 才会真正执行操作.
- parallelStream()方法的操作可以并行执行.
- parallelStream()方法只能在单线程中执行, 它只能处理一个分区的数据.
- parallelStream()方法的并行处理能力取决于CPU的核心数、JVM的内存、磁盘IO、网络IO、其他资源的限制.

<br>

### stream()和parallelStream()的区别

- stream()和parallelStream()的区别主要在于并行处理的能力.
- stream()方法返回的是顺序流, 它只能在单线程中执行, 它只能处理一个分区的数据.
- parallelStream()方法返回的是并行流, 它可以在多个线程中执行, 它可以处理整个集合的数据.

<br>

### 踩坑

1. 问题汇总

   - ParallelStreams()使用JVM默认的forkJoin框架的线程池由当前线程去执行并行操作
   - 阻塞操作: 调用第三方API时, 由于响应时间较长, 会导致线程阻塞, 影响整体性能.
   - 线程池耗尽: ForkJoinPool.common() 的线程池可能会被耗尽, 导致后续任务性能下降.
   - 并行流的影响: 使用 ParallelStream() 时, 长时间运行的函数或阻塞操作会影响整个程序的性能, 导致其他部分的执行变得不可预测.

2. 解决方案

   - (1) 异步处理: 使用 CompletableFuture 或其他异步编程模型来处理第三方API的调用, 避免阻塞主线程或其他工作线程

    ```java
    List<CompletableFuture<Response>> futures = apiUrls.stream()
        .map(url -> CompletableFuture.supplyAsync(() -> callApi(url), executor))
        .collect(Collectors.toList());
    
    CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
    ```

   - (2) 自定义线程池: 为不同的任务类型创建不同的线程池, 避免所有任务共享同一个线程池导致的资源竞争问题.

    ```java
    ExecutorService apiExecutor = Executors.newFixedThreadPool(10);
    ExecutorService computeExecutor = Executors.newFixedThreadPool(5);
    
    // 使用apiExecutor处理API调用
    CompletableFuture<Response> apiFuture = CompletableFuture.supplyAsync(() -> callApi(url), apiExecutor);
    
    // 使用computeExecutor处理计算任务
    CompletableFuture<Result> computeFuture = CompletableFuture.supplyAsync(() -> computeData(data), computeExecutor);
    ```

   - (3) ManagedBlocker: 在某些情况下, 可以使用 ManagedBlocker 来帮助 ForkJoinPool 管理阻塞操作, 确保线程池的线程不会被完全阻塞.

    ```java
    class ApiBlocker implements ManagedBlocker {
        private volatile boolean done = false;
        private final String url;
        private Response response;
    
        public ApiBlocker(String url) {
            this.url = url;
        }
    
        @Override
        public boolean block() {
            response = callApi(url);
            done = true;
            return true;
        }
    
        @Override
        public boolean isReleasable() {
            return done;
        }
    
        public Response getResponse() {
            return response;
        }
    }
    
    ForkJoinPool.managedBlock(new ApiBlocker(url));
    ```

   - (4) 避免长时间运行的函数: 在 ParallelStream() 中尽量避免使用长时间运行的函数, 或者将这些函数拆分成更小的任务, 以减少对线程池的影响.

<br>

### 总结

- stream()方法和parallelStream()方法都是Java 8引入的新特性,它们都是用来处理集合、数组、I/O流、生成器等的高级操作.
- stream()方法只能在单线程中执行,它只能处理一个分区的数据,它的操作是顺序执行的,它的操作是延迟执行的,只有调用终止操作,才会真正执行操作.
- parallelStream()方法可以在多个线程中执行,它可以处理整个集合的数据,它的操作是并行执行的,它的操作是延迟执行的,只有调用终止操作,才会真正执行操作.
- parallelStream()方法的并行处理能力取决于CPU的核心数、JVM的内存、磁盘IO、网络IO、其他资源的限制.
- 正确使用stream()方法和parallelStream()方法,可以提高代码的执行效率,并行处理集合数据,提高处理速度.
- 通过异步处理、自定义线程池和合理使用 ManagedBlocker, 可以有效避免 ForkJoinPool 和 ParallelStream
  带来的性能问题. 根据具体的应用场景, 选择合适的解决方案可以显著提升程序的性能和稳定性. 