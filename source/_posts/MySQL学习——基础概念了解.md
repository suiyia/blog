title: MySQL学习——基础概念了解
author: Answer
tags:
  - MySQL
categories:
  - MySQL
date: 2019-10-10 21:44:00
---
### MySQL 主从复制原理

https://zhuanlan.zhihu.com/p/50597960

MySQL主从复制涉及到三个线程

- 主节点 binary log dump 线程

当从节点连接主节点时，主节点会创建一个log dump 线程，用于发送bin-log的内容。在读取bin-log中的操作时，此线程会对主节点上的bin-log加锁，当读取完成，甚至在发动给从节点之前，锁会被释放。
    
- 从节点 I/O线程

当从节点上执行`start slave`命令之后，从节点会创建一个I/O线程用来连接主节点，请求主库中更新的bin-log。I/O线程接收到主节点binlog dump 进程发来的更新之后，保存在本地relay-log中。
    
- 从节点SQL线程

SQL线程负责读取relay log中的内容，解析成具体的操作并执行，最终保证主从数据的一致性。

### binlog 记录格式

- 基于SQL语句，只需要记录会修改数据的sql语句到binlog中，减少了binlog日质量，节约I/O，提高性能。缺点是在某些情况下，会导致主从节点中数据不一致（比如sleep(),now()等）。
    
- 基于行，只记录哪条数据被修改了，修改成什么样。优点是不会出现某些特定情况下的存储过程、或者函数、或者trigger的调用或者触发无法被正确复制的问题。缺点是会产生大量的日志，尤其是修改table的时候会让日志暴增,同时增加bin log同步时间。也不能通过bin log解析获取执行过的sql语句，只能看到发生的data变更。
    
- 混合模式，一般的复制使用STATEMENT模式保存到binlog，对于STATEMENT模式无法复制的操作则使用ROW模式来保存，MySQL会根据执行的SQL语句选择日志保存方式。

### 日志类型

- 错误日志，记录出错信息，也记录一些警告信息或者正确的信息

- 查询日志，记录所有对数据库请求的信息，不论这些请求是否得到了正确的执行

- 慢查询日志，设置一个阈值，将运行时间超过该值的所有SQL语句都记录到慢查询的日志文件中。

- 二进制日志，记录对数据库执行更改的所有操作

- 中继日志，事务日志


### 数据类型

（https://juejin.im/entry/5b57ec015188251aa8292a69）

- 整数类型，包括TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT，分别表示1字节、2字节、3字节、4字节、8字节整数。
    
- 实数，包括FLOAT、DOUBLE、DECIMAL。DECIMAL可以用于存储比BIGINT还大的整型，能存储精确的小数。而FLOAT和DOUBLE是有取值范围的，并支持使用标准的浮点进行近似计算。计算时FLOAT和DOUBLE相比DECIMAL效率更高一些，DECIMAL你可以理解成是用字符串进行处理。
    
- 字符串类型，包括 VARCHAR、CHAR、TEXT、BLOB。VARCHAR 用于存储可变长字符串，它比 CHAR 定长类型更节省空间；当CHAR值被存储时，它们被用空格填充到特定长度，检索CHAR值时需删除尾随空格。 
    
- 枚举类型，把不重复的数据存储为一个预定义的集合。有时可以使用ENUM代替常用的字符串类型。ENUM存储非常紧凑，会把列表值压缩到一个或两个字节。ENUM在内部存储时，其实存的是整数。尽量避免使用数字作为ENUM枚举的常量，因为容易混乱。排序是按照内部存储的整数
    
- 日期时间类型，尽量使用timestamp，空间效率高于datetime，用整数保存时间戳通常不方便处理。如果需要存储微妙，可以使用bigint存储。

### CHAR和VARCHAR的区别？

- CHAR和VARCHAR类型在存储和检索方面有所不同

- CHAR列长度固定为创建表时声明的长度，长度值范围是1到255

- 当CHAR值被存储时，它们被用空格填充到特定长度，检索CHAR值时需删除尾随空格。

### MYSQL中varchar(10)和int(10)的区别 TODO

### SQL 语句分类

- DDL：数据定义语言（create drop alter）
- DML：数据操作语句（insert update delete）
- DQL：数据查询语句（select ）
- DCL：数据控制语句，进行授权和权限回收（grant revoke）
- TPL：数据事务语句（commit collback savapoint）

### 数据库范式

减少数据冗余、查询需要多表关联

- 第一范式：属性具有原子性，不可再分解。

- 第二范式：记录具有唯一标识，即实体唯一性。通常需要为表加上一个列， 以存储各个实例的惟一标识。

- 第三范式：任何字段不能由其它字段派生而来，要求字段没有冗余。第三范式具有如下特征：1， 每一列只有一个值。2， 每一行都能区分。3， 每一个表都不包含其他表已经包含的非主关键字信息。


### MyISAM表格将在哪里存储，并且还提供其存储格式?
每个MyISAM表格以三种格式存储在磁盘上：

- “.frm”文件存储表定义

- 数据文件具有“.MYD”(MYData)扩展名

- 索引文件具有“.MYI”(MYIndex)扩展名


# 其它

[一条SQL语句在MySQL中是如何执行的](https://juejin.im/post/5c91b5d7e51d456f957ebd10)

[MySQL 主从复制原理](https://zhuanlan.zhihu.com/p/50597960)

[MySQL经典面试题](https://www.cnblogs.com/panwenbin-logs/p/8366940.html)

