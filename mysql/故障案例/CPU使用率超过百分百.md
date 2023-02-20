## 因素

CPU100%三大因素：慢SQL，锁，资源

1. 慢SQL

数据库出现慢SQL，慢SQL堆积，CPU上下文切换非常厉害，CPU很容易被打满。所以当慢SQL变慢时，会把数据库带慢。

1. 锁

出现锁同样会导致CPU100%。

1. 资源

如果配置比较低，但需求较高，去做资源升级。



## 解决方案

1. 慢SQL问题

通过优化索引，子查询，隐士转换，分页改写等优化；

用户可以登录到rds，通过**show processlist**查看当前正在执行的sql，当执行完show processlist后出现大量的语句，通常其状态出现sending data，Copying to tmp table，Copying to tmp table on disk，Sorting result, Using filesort 都是sql有性能问题；

**sending data**：sql正在从表中查询数据，如果查询条件没有适当的索引，则会导致sql执行时间过长；

**Copying to tmp table on disk**：出现这种状态，通常情况下是由于临时结果集太大，超过了数据库规定的临时内存大小，需要拷贝临时结果集到磁盘上，这个时候需要用户对sql进行优化；

**Sorting result, Using filesort**：出现这种状态，表示sql正在执行排序操作，排序操作都会引起较多的cpu消耗，通常的优化方法会添加适当的索引来消除排序，或者缩小排序的结果集

2. 锁等待问题

通过设计开发和资源运维优化锁等待；

3. 资源问题

通过参数优化，弹性升级。读写分离，数据库拆分等方式优化；  