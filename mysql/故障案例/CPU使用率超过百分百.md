rds 实例cpu 超过100%，通常这种情况都是由于sql 性能问题导致的

案例：cpu 超过100%，实例偶尔出现卡住的现象

**原理：**cpu 消耗过大通常情况下都是有慢sql 造成的，这里的慢sql 包括全表扫描，扫描数据量过大，内存排序，磁盘排序，锁争用等待等；

**表现现象：**sql 执行状态为：sending data，Copying to tmp table，Copying to tmp

table on disk，Sorting result，locked;

**解决方法：**用户可以登录到rds，通过show processlist查看当前正在执行的sql，当执行完show processlist后出现大量的语句，通常其状态出现sending data，Copying to tmp table，Copying to tmp table on disk，Sorting result, Using filesort 都是sql有性能问题；

A.sending data表示：sql正在从表中查询数据，如果查询条件没有适当的索引，则会导致sql执行时间过长；

B.Copying to tmp table on disk：出现这种状态，通常情况下是由于临时结果集太大，超过了数据库规定的临时内存大小，需要拷贝临时结果集到磁盘上，这个时候需要用户对sql进行优化；

C.Sorting result, Using filesort：出现这种状态，表示sql正在执行排序操作，排序操作都会引起较多的cpu消耗，通常的优化方法会添加适当的索引来消除排序，或者缩小排序的结果集