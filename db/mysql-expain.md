###1、概述
---
>执行计划的查看是sql语句调优时依据的一个重要依据，mysql的执行计划查看相对oracle简便很多，功能也相对简单很多的SQL语句都不能直接查看。

>执行计划加上慢查询日志组成了mysql调优过程的一组调优利器，当数据库稳定过后参数的调优是很少的一部分，80%以上的调优都会是SQL调优

###2、Explain语法
---
Explain语法

```EXPLAIN  SELECT ……
变体：
1. EXPLAIN EXTENDED SELECT ……
将执行计划“反编译”成SELECT语句，运行SHOW WARNINGS 可得到被MySQL优化器优化后的查询语句 
2. EXPLAIN PARTITIONS SELECT ……
用于分区表的EXPLAIN```

