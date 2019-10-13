title: SLF4J LOG4J Logback logging 日志框架概念学习
author: Answer
date: 2019-10-02 10:02:32
tags:
---
今天在群里看到别人分享的一个面试题，常用的日志框架有哪些？他们有什么区别和联系？

看到这个问题的刹那，脑子一片空白，平时只是拿来用用，并没有当回事，现在才知道不懂的是真的很多，每天多学点知识吧，加油！

### 日志框架



- 接口


- [**SLF4J**](https://www.slf4j.org/)（Simple Logging Facade for Java），定义了各种日志框架（java.util.logging, logback, log4j 等）的**抽象**，是 Java 日志输出的**接口**。需引入 slf4j-api-xxx.jar

- [**log4j 2.X**](https://logging.apache.org/log4j/2.x/)，Apache 表示它是 log4j 的升级版，和 SLF4J 一样，依赖 log4j-api 包可以定义日志输出标准接口，依赖 log4j-core 包，将定义日志的实际输出格式

- **Simple**，SLF4J 的简单实现，需引入 slf4j-simple-xxx.jar

- **java.util.logging**，jdk 自带的日志工具类，它通过 getLogger 获取日志对象、setLevel 定义日志级别、Handler 定义日志输出方式 （输出到文件、控制台、网络流）、Formatter 定义日志输出样式。需引入 slf4j-jdk14-xxx.jar

- [**Logback**](https://logback.qos.ch/)，官网表示它可作为 log4j 的 successor（继承者），比 log4j 更好！它包括 3 个模块：
  - logback-core，下面两个模块的基础依赖
  - logback-classic，对 log4j 进行了显著地改进，实现了 SLF4J API，方便随时切换日志框架。
  - logback-access，与 Servlet 容器（Tomcat、Jetty）进行集成，提供 HTTP 访问日志功能。



- **NOP**，A direct NOP (no operation) implementation of Logger，具体干嘛不清楚。需引入 slf4j-nop-xxx.jar

- [**JCL**](http://svn.apache.org/repos/asf/commons/proper/logging/tags/LOGGING_1_0_3/usersguide.html)(Jakarta Commons Logging)，又叫 Apache Commons Logging，和 SLF4J 类似，提供日志框架抽象，方便开发者使用不同的日志实现类框架。

他们之间的**联系**就是：使用 slf4j **定义日志输出模板**，然后使用具体的日志框架（java.util.logging, logback, log4j等）**定义日志具体输出的形式**。

### 使用
1. 首先引入 slf4j-api-xxx.jar 包，通过 Logger 与 LoggerFactory 输出日志，这样日志输出的模板就定义好了，两行代码即可输出相应的日志。
```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}

console output：
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.

```
2. 但是仅仅引入 slf4j-api 的 jar 包还不行，会提示没有绑定具体日志框架实现。
3. 还需要引入**绑定** slf4j 的日志框架，定义日志输出的具体形式，常用的日志实现框架有以下几种：
   - slf4j 自带的 slf4j-simple-xxx.jar
 
   - logback
      ```xml
      <dependency> 
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.0.13</version>
      </dependency>
      ```
    - log4j
      ```
      <dependency> 
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.8.0-beta2</version>
      </dependency>
      ```
    - java.util.logging
      ```
      <dependency> 
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-jdk14</artifactId>
        <version>1.8.0-beta2</version>
      </dependency>
      ```

#### SLF4J 使用注意事项
- 不绑定具体的日志框架

若程序只加入了 slf4j-api 依赖，而没有设置具体的日志框架，那么 SLF4J 会发出警告，并且忽略所有的日志打印请求

- 版本兼容

slf4j-api 不同版本是互相兼容的，不同系统之间的 slf4j-api 版本不同并不会造成冲突；但是其实现类是可能存在冲突的，例如使用 slf4j-api-1.0.jar 和 slf4j-simple-1.0.jar 是没有问题的，但是使用 slf4j-simple-2.0.jar 可能存在问题。在引入相关依赖的时候，要保证 slf4j-api 的版本和其实现日志框架的版本一致！

- 格式输出

slf4j 支持占位符输出
```java
logger.debug("Temperature set to {}. Old temperature was {}.", t, oldT);
```

### 博客参考
[SLF4J和Logback和Log4j和Logging的区别与联系](https://blog.csdn.net/qq_32625839/article/details/80893550)

### 总结
- SLF4J 是个日志抽象框架，定义基本的日志输出功能，必须引入。SLF4J 可以与其它的日志框架相结合，输出自己想要的日志格式。
- 看文档是个不错的选择，英文刚开始看可能有些浮躁，静下心来慢慢看，结合翻译一定能看出点东西
- 看了几篇博客和文档，算是有了一个大概，内心感觉踏实了一点，至少知道自己在做什么，最坏的情况的就是不知道自己不知道，还在那里悠然自乐。下次再学习下 log4j 在 Spring 中的配置。