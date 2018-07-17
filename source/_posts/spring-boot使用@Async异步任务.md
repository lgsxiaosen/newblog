---
title: spring boot使用Async异步任务
date: 2018-07-17 11:04:05
tags:
- java
- spring boot
- Async
- 异步
category:
- 技术文章项目
---



github项目地址：https://github.com/lgsdaredevil/asyncTest



#### 开启异步任务

在应用主类中添加@EnableAsync注解
![在应用主类中添加注解](https://raw.githubusercontent.com/lgsdaredevil/newblog/resource-newblog/source/favicons/article/QQ20180717111459.png)


#### 写异步任务方法
```java
	@Async
    public Future<String> ansync(String name){
        try {
            Thread.sleep(10000);
            logger.info("这里是异步方法");
            logger.info("传过来的名字是：" + name);
            name = "修改的名字";
            logger.info("修改后的名字是：" + name);
            return new AsyncResult<>("name: " + name);
        }catch (Exception e){
            return new AsyncResult<>("异常");
        }
    }
```
#### 调用异步方法
* 1、用Future获取返回值
```java
public String requestAnsync(String name){
        try {
            Long start = System.currentTimeMillis();
            Future<String> result = ansync(name);
            if (result.isDone()){
                name = result.get();
                logger.info("异步方法结束，名字改为：" + name);
            }
            Long end = System.currentTimeMillis();
            logger.info("耗时：" + (int)(end-start));
            return "hello " + name;
        }catch (Exception e){
            logger.error("异常");
            return "异常";
        }
    }
```
返回值，若想获取到返回值，应该轮询方法获取，否则若果没有isDone则不会走下面的方法，或者可以使用CompletableFuture：
```
2018-07-17 11:31:55.390  INFO 5232 --- [nio-8080-exec-6] c.e.async.service.AsyncTestService       : 耗时：0
2018-07-17 11:32:05.394  INFO 5232 --- [cTaskExecutor-3] com.example.async.service.AsyncTest      : 这里是异步方法
2018-07-17 11:32:05.394  INFO 5232 --- [cTaskExecutor-3] com.example.async.service.AsyncTest      : 传过来的名字是：ling
2018-07-17 11:32:05.394  INFO 5232 --- [cTaskExecutor-3] com.example.async.service.AsyncTest      : 修改后的名字是：修改的名字
```

如果使用future.get()方法会阻塞线程直到拿到结果。
* 2、不使用future.get()方法，异步方法不使用Future返回
```java
    @Async
    public void noReturnAsync(String name){
        try {
            Thread.sleep(10000);
            logger.info("这里是异步方法");
            logger.info("传过来的名字是：" + name);
            name = "修改的名字";
            logger.info("修改后的名字是：" + name);
        }catch (Exception e){
        }
    }
```
调用异步的方法
```java
public String noReturn(String name){
        Long start = System.currentTimeMillis();
        asyncTest.noReturnAsync(name);
        Long end = System.currentTimeMillis();
        logger.info("耗时：" + (int)(end-start));
        return "hello " + name;
    }
```

#### 注意的地方：
如果异步方法变成阻塞的同步方法，可能原因是异步方法和普通的调用方法在同一个类中，解决方法是将异步方法单独放到一个类中。
产生原因：spring对@Transactional注解时也有类似问题，spring扫描时具有@Transactional注解方法的类时，是生成一个代理类，由代理类去开启关闭事务，而在同一个类中，方法调用是在类体内执行的，spring无法截获这个方法调用。
具体参见：[Spring Boot使用@Async实现异步调用](https://www.cnblogs.com/shihaiming/p/7825204.html)




