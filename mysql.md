# mysql

1、InnoDB与MyISAM的区别

2、MySQL事务得四大特性以及实现原理

- 原子性： 事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行。
- 一致性： 指在事务开始之前和事务结束以后，数据不会被破坏，假如A账户给B账户转10块钱，不管成功与否，A和B的总金额是不变的。
- 隔离性： 多个事务并发访问时，事务之间是相互隔离的，即一个事务不影响其它事务运行效果。简言之，就是事务之间是进水不犯河水的。
- 持久性： 表示事务完成以后，该事务对数据库所作的操作更改，将持久地保存在数据库之中。

事务ACID特性的实现思想

- 原子性：是使用 undo log来实现的，如果事务执行过程中出错或者用户执行了rollback，系统通过undo log日志返回事务开始的状态。
- 持久性：使用 redo log来实现，只要redo log日志持久化了，当系统崩溃，即可通过redo log把数据恢复。
- 隔离性：通过锁以及MVCC,使事务相互隔离开。
- 一致性：通过回滚、恢复，以及并发情况下的隔离性，从而实现一致性。

3、主从复制原理

- 步骤一：主库的更新事件(update、insert、delete)被写到binlog
- 步骤二：从库发起连接，连接到主库。
- 步骤三：此时主库创建一个binlog dump线程，把binlog的内容发送到从库。
- 步骤四：从库的I/O线程读取主库传过来的binlog内容并写入到relay log
- 步骤五：从库的SQL线程从relay log里面读取内容，从Exec_Master_Log_Pos位置开始执行读取到的更新事件，将更新内容写入到slave的db

4、主从复制类型

mysql主从复制有异步复制、半同步复制、GTID复制等。

### 异步复制

默认复制模式，主库将数据写到binlog就直接返回给客户端更新成功，**不用关心 Slave 是否接收到二进制日志**

有点风险（如果还没来得及同步到从库就会丢失数据），但**性能好**（从库复制不会影响到主库写数据）

如果业务对于数据一致性要求不高，当发生故障时，能容忍数据的丢失，甚至大量的丢失，推荐用异步复制，这样性能最好

### 半同步复制

同步复制：主库执行完客户端提交的事务后，需要等**所有从库都执行完这一事务**后才返回给客户端执行成功。性能有所影响。

半同步复制：介于异步和同步之间，主库执行完客户端提交的事务后要等待**至少一个从库接收到binlog并写入到relaylog中**才返回给客户端执行成功。

半同步，在主库的dump线程增加了一个ack机制，不但要发送binlog，还要接收从库的ack，当出现异常将自动降级为异步复制，直到修复后变成半同步。

风险：

客户端提交事务后，主库在等待从库ack时宕机了，会有两种情况：

- 事务还没发生到从库：客户端收到失败后会重新提交事务，重新提交是在新的主库执行，所以会执行成功，如果之前的主库恢复了，会以从库身份加入集群，这时候之前的事务会被执行两次：第一次是此机器作为主库时执行，第二次是作为从库后从主库同步过来

- 事务已经同步到从库：客户端收到失败后重新提交，事务会在从库执行两次

解决：

5.7版本开始增加新的半同步方式（5.7.2默认）：从库的事务ack后，才提交主库事务

依然有风险：如果主库等待从库同步成功的过程主库就挂了，主库事务就提交失败，但从库已经把binlog写入relaylog，这时从库数据就多了，但多了一半不严重。

半同步复制模式参数：

```mysql
show variables like '%Rpl%';
```

### GTID复制

5.6开始推出GTID模式（全局事务ID）

GTID由server_uuid（实例唯一标识）和transactionld（该mysql执行事务的数量）组成

```mysql
show master status;
```

原理：

```text
从库把自己执行过的GTID和获取的GTID都传给主库，主库发送缺少的GTID给从库，让从库补全数据。当主库挂了，会把同步最成功的从库提升为主库。

master更新数据时，在事务前生产GTID，一同记录到binlog
slave把变更的binlog写入relaylog
sql线程从relaylog获取GTID，然后对比slave的binlog是否有记录
如果有记录说明该GTID事务已经执行，slave会忽略
如果没有记录，slave从relaylog执行该GTID事务，并记录到binlog
```

5、主从同步延迟原因及解决办法

6、优化mysql

配置优化：

sort/join/read/read rnd buffer 4-16M就满足大部分需求了

#双1标准

sync_binlog=1   什么时候刷新binlog到磁盘，每次事务commit
innodb_flush_log_at_trx_commit=1

long_query_time = 0.1

#交互连接和程序连接的超时时间

interactive_timeout = 600
wait_timeout = 600

innodb_thread_concurrency=0 不限制innodb线程

innodb_lock_wait_timeout = 10 行锁等待时间

#innodb的iops，设置为硬盘iops的上限

innodb_io_capacity = 4000
innodb_io_capacity_max = 8000

#redo log大小

innodb_log_file_size = 1G #如果线上环境的TPS较高，建议加大至1G以上，如果压力不大可以调小
innodb_log_files_in_group = 3

7、优化RDS

连接数、内存不能修改，涉及主备数据安全的参数比如innodb_flush_log_at_trx_commit、sync_binlog、gtid_mode、semi_sync、binlog_format等不能修改

open_files_limit MySQL实例能够**同时打开使用的文件句柄数目**，RDS在起初初始化实例的时候设置的open_files_limit为8192，当打开的表数目超过该参数则会导致所有的数据库请求报错误。最大为65535

back_log 如果前端应用有大量的短连接请求到达数据库，MySQL 会限制此刻新的连接进入请求队列，如果**等待的连接数量**超过back_log，则将不会接受新的连接请求

innodb_autoinc_lock_mode 用于控制自增主键的锁机制，建议将参数设置改为2，则表示所有情况插入都使用轻量级别的mutex锁(只针对row模式)，这样就可以避免auto_inc的死锁，同时在INSERT … SELECT 的场景下会提升很大的性能(注意该参数设置为2，binlog的格式需要设置为row)。

tmp_table_size 如果应用中有很多group by/distinct等语句，同时数据库有足够的内存，可以增大tmp_table_size(max_heap_table_size)的值，以此来提升查询性能。

max_statement_time 如果用户希望控制数据库中SQL的执行时间，则可以开启该参数，单位是毫秒。

rds_threads_running_high_watermark 该参数常常在秒杀或者大并发的场景下使用，对数据库具有较好的保护作用。

8、RDS备份

备份在备实例执行，不占用主实例CPU，不影响主实例性能。

可使用命令行或图形界面进行逻辑数据备份。仅限通过 RDS 管理控制台 或 OPEN API 进行物理备份。

恢复：

恢复到一个新实例，验证数据后，再将数据迁回原实例

9、binlog类型

STATEMENT：基于SQL语句的复制，每一条会修改数据的sql语句会记录到binlog中。该模式下产生的binlog日志量会比较少，但可能导致主从数据不一致。

ROW：基于行的复制，不记录每一条具体执行的SQL语句，仅需记录哪条数据被修改了，以及修改前后的样子。该模式下产生的binlog日志量会比较大，但优点是会非常清楚的记录下每一行数据修改的细节，主从复制不会出错。

Mixed：混合模式复制，以上两种模式的混合使用，一般的复制使用STATEMENT模式保存binlog，对于STATEMENT模式无法复制的操作使用ROW模式保存binlog，MySQL会根据执行的SQL语句选择日志保存方式。
