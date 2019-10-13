title: MySQL学习——索引
author: Answer
tags:
  - MySQL
categories:
  - MySQL
date: 2019-10-10 21:38:00
---
# 索引

索引是一种数据结构,可以帮助我们快速的进行数据的查找。InnoDB 存储引擎的默认索引实现为 B+ 树索引。

## 索引分类

- 普通索引（index）：最基本的索引，没有任何约束限制。
    
- 唯一索引（unique）：和普通索引类似，但是具有唯一性约束，允许有空值，一个表可以有多个唯一索引。
    
- 主键索引（primary key）：特殊的唯一索引，不允许有空值，保证实体完整性，一个表只能有一个主键。
    
- 联合索引：将多个列组合在一起创建索引，可以覆盖多个列。（也叫复合索引，组合索引）
    
- 全文索引：主要用于查找文本中的关键字，并不是直接与索引中的值进行比较。fulltext 更像是一个搜索引擎，配合 match against 操作使用，而不是一般的 where 语句加 like。
    
    
## 索引实现

MyISAM 和 InnoDB 都使用 B+Tree 实现。MyISAM 叶子节点存放数据记录的地址，要找到该数据还需要去该地址寻找。InnoDB 叶子节点就是数据本身，数据表的主键作为索引的 key。

聚簇索引，MySQL索引是用一种叫做聚簇索引的数据结构实现的，是一种数据存储方式，它实际上是在同一个结构中保存了 B+ 树索引和数据行，InnoDB 表是按照聚簇索引组织的（类似于Oracle的索引组织表）。

聚簇索引的叶节点就是数据节点，而非聚簇索引的叶节点仍然是索引节点，并保留一个链接指向对应数据块。

## explain 用法字段含义(分析是否有用到索引)

https://www.sunjs.com/article/detail/99169a7375834c158409036934f10fab.html

- id 表示一个查询中各个子查询的执行顺序，id相同执行顺序由上至下。 id不同，id值越大优先级越高，越先被执行。id为 null 时表示一个结果集，不需要使用它查询，常出现在包含union等查询语句中。

- select_type 表示查询中每个 select 子句的类型：
    
    - SIMPLE 不包含任何子查询或union等查询
    
    - PRIMARY 包含子查询最外层查询就显示为 PRIMARY
    
    - SUBQUERY 在select或 where字句中包含的查询
    
    - DERIVED from字句中包含的查询
    
    - UNION 出现在union后的查询语句中
    
    - UNION RESULT 从UNION中获取结果集

- table 查询的数据表，当从衍生表中查数据时会显示 x ，表示对应的执行计划id

- partitions 表分区、表创建的时候可以指定通过那个列进行表分区。

- type 表示MySQL在表中找到所需行的方式，又称「访问类型」
    
    - ALL 扫描全表数据
    
    - index 遍历索引
    
    - range 索引范围查找
    
    - index_subquery 在子查询中使用 ref
    
    - unique_subquery 在子查询中使用 eq_ref
    
    - ref_or_null 对Null进行索引的优化的 ref
    
    - fulltext 使用全文索引
    
    - ref 使用非唯一索引查找数据
    
    - eq_ref 在join查询中使用PRIMARY KEYorUNIQUE NOT NULL索引关联。
    
- possible_keys 可能使用的索引，但不一定被查询使用

- key 显示MySQL在查询中实际使用的索引，若没有使用索引，显示为NULL; 查询中若使用了覆盖索引(覆盖索引：索引的数据覆盖了需要查询的所有数据)，则该索引仅出现在key列表中

- key_len 表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度

- ref 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

- rows 返回估算的结果集数目，并不是一个准确的值。

- filtered 

- Extra 包含不适合在其他列中显示但十分重要的额外信息
    
    - Using index 使用覆盖索引 
    
    - Using where 使用了用where子句来过滤结果集 
    
    - Using filesort 使用文件排序，使用非索引列进行排序时出现，非常消耗性能，尽量优化。 
    
    - Using temporary 使用了临时表


## 索引失效情况

- % 开头的 like 模糊匹配

- or 前后语句没有同时使用索引

- 数据类型出现隐式转化（字符串列查询没有使用引号）

- 索引列进行运算