title: 日志框架学习及Log4j 2 概念整理
author: Answer
tags:
  - Log4j
categories: []
date: 2019-09-04 14:52:00
---
# 日志框架分类

日志框架按照功能可以分为日志**接口**、日志**实现**两部分。

编写程序时，推荐使用**日志接口**的 API 进行方法调用，然后使用对应的**日志实现框架**打印日志。

常用日志框架的使用方式为： Log4j2-api（接口） + Log4j2-core（实现）。SLF4J（接口）+ 其它日志框架实现。  

- 日志接口，日志的接口规范，它对用户提供了统一的日志接口，屏蔽了不同日志组件的差异。

    - [Apache Commons Logging Component](https://commons.apache.org/proper/commons-logging/)，2014 年之后文档没有再更新

    - [SLF4J（Simple Logging Facade for Java）](https://www.slf4j.org/)，定义了各种日志框架（java.util.logging, logback, log4j 等）的**抽象**，是 Java 日志输出的标准接口。需引入 slf4j-api-xxx.jar
    
        - slf4j-api 不同版本是互相兼容的，不同系统之间的 slf4j-api 版本不同并不会造成冲突；但是其实现类是可能存在冲突的，例如使用 slf4j-api-1.0.jar 和 slf4j-simple-1.0.jar 是没有问题的，但是使用 slf4j-simple-2.0.jar 可能存在问题。在引入相关依赖的时候，要保证 slf4j-api 的版本和其实现日志框架的版本一致！
         
    - [Log4J2 api](https://logging.apache.org/log4j/2.x/)，log4j 2-api 包和 slf4J 类似，定义日志输出接口规范，具体日志输出形式根据依赖的日志实现 jar 包确定

- 日志实现，定义具体日志打印内容

    - JDKLog，java.util.logging.Logger，jdk 自带的日志工具类，它通过 getLogger 获取日志对象、setLevel 定义日志级别、Handler 定义日志输出方式 （输出到文件、控制台、网络流）、Formatter 定义日志输出样式。需引入 slf4j-jdk14-xxx.jar
    
    - [LOGBack](https://logback.qos.ch/)，继承于 Log4J，官网表示它可作为 log4j 的 successor（继承者），比 log4j 更好！它包括 3 个模块：
		- logback-core，下面两个模块的基础依赖
  		- logback-classic，对 log4j 进行了显著地改进，实现了 SLF4J API，方便随时切换日志框架。
  		- logback-access，与 Servlet 容器（Tomcat、Jetty）进行集成，提供 HTTP 访问日志功能。
    
    - [Log4J](http://logging.apache.org/log4j/1.2/)，2015 年开始已经停止维护

    - [Log4J2 core](https://logging.apache.org/log4j/2.x/) Log4J 升级版，2019-08-06 版本更新至 2.12.1，API 相关用法不兼容 1.X 版本


# Log4j 2 内置概念

- Markers：为某条日志添加标志位，使用 %marker 进行输出

```
public void test(){
    private static final Marker SQL_MARKER = MarkerManager.getMarker("SQL");
    Marker markerA = MarkerManager.getMarker("Marker A");
    Marker markerB = MarkerManager.getMarker("Marker B");
    logger.debug(markerA,"your name is {} ","Jack");
    logger.debug(markerB,"your name is {} ","Ma");
}

log4j.xml：
<PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M %marker - %msg%xEx%n"/>

Console Out：
10:14:02.639 DEBUG FlowTracing 60 test Marker A - your name is Jack 
10:14:02.642 DEBUG FlowTracing 61 test Marker B - your name is Ma 
```

- Lookups：在配置文件中添加变量、使用变量

- Appenders：定义日志事件输出方式（日志文件名称、路径、什么级别日志才能输出、文件大小等等），其实就是文件流输出定义。
    - AsyncAppender 用另外的线程记录日志，默认使用 ArrayBlockingQueue 队列进行日志记录，当发生异常时，默认会忽略异常，记得手动开启，并使用FailoverAppender 对异常情况进行补充打印输出
    - ConsoleAppender，控制台输出
    - FailoverAppender，当主 Appender 失败时，它将起作用
    - FileAppender，文件输出，当多个程序对同一文件进行输出时，各自程序之间的配置也不受影响
    - JDBCAppender，直接写到数据库
    - HttpAppender，http 请求发送到另外的服务接收，如果服务返回的不是 2XX，将会抛出异常。
    - KafkaAppender，发送到 Kafka
    - RollingFileAppender，可以滚动的文件输出，包含触发规则（触发滚动条件：时间、大小）、滚动规则（怎么进行更新：文件数量），默认日志文件个数为 7 个。
    - [等等](https://logging.apache.org/log4j/2.x/manual/appenders.html)

- Layouts：Appender 使用定义好的 LayOut 对 LogEvent 进行格式化输出，
    - CSV Layouts，以 Comma Separated Value (CSV) 形式输出
    - JSON Layout，JSON 格式输出
    - Pattern Layout，用特殊的字符输出对应的内容，定制化强
    - XML Layout
    - Location Infomation，当日志中需要日志所在类、方法、代码行数这些信息时，会花费 1.3-5 倍的时间，相比不需要 Location Infomation 的场景。并且在使用异步的 Appender 时，默认的 Location Infomation 默认关闭，如果开启，会比不开启花费 30-100 倍的性能消耗

- Filters：决定哪些事件可以输出，哪些事件不能输出。过滤结果有 3 个结果，ACCEPT（允许）, DENY（拒绝输出）or NEUTRAL（中立、默认）。
    - BurstFilter，当每秒日志输出事件速率大于一定值，将会抛弃掉级别低的日志
    - CompositeFilter，直接在 configuration 级别添加多个过滤器
    - DynamicThresholdFilter，根据一些属性值去动态修改该条日志的输出级别
    - MapFilter，如果 LogEvent 中的 MapMessage 内包含特定属性，就可以选择过滤
    - MarkerFilter、NoMarkerFilter，对应定义或者未定义 Marker 的事件进行过滤
    - RegexFilter，正则过滤 formatted or unformatted message
    - ThreadContextMapFilter ，对 ThreadContextMap 中的内容进行过滤
    - ThresholdFilter，对 Level 进行过滤
    - TimeFilter，根据某个时间点、时间段进行过滤


# 3. Log4J 2 Java API 用法

- Flow Tracing

	日志跟踪

- Markers（标记）

	对于有特定标识需求的地方，可以使用 Marker 进行标记，并且用 %marker 进行打印输出。

- ThreadContext

	Mapped Diagnostic Context（MDC）的实现，类似 request，可以存放 key-value 键值对，也可以销毁，使用 %X{key} 输出。

	the MDC is a map which stores the context data of the particular thread where the context is running. 


  ```
  %X{id} 输出 context map 中 key = id 的值
  %X 输出 context map 所有值

  public void test(){
      ThreadContext.put("id", "321"); // Add the fishtag;
      ThreadContext.put("name", "996"); // Add the fishtag;
      logger.debug("Message 1");
      logger.debug("Message 2");
      ThreadContext.clearAll();
  }
  ```


# 其他

log4j-slf4j-impl 是 slf4j 转接到 log4j 2 的日志输出框架。

slf4j-log4j12 是 slf4j 转接到 log4j 1.x 的日志输出框架 ，而 log4j 1.x 已经在 2015 年 8 月就停止更新了。

log4j 官方建议升级到 log4j 2