# mysql配置

## 参数详解

客户端:

```shell
[client]
#默认链接的端口
port=3306
#默认链接的socket的位置
socket=/var/lib/mysql.sock
#默认编码格式
default-character-set=utf8mb4
```

基础设置:

```shell
[mysqld]
#MySQL启动用户
user = mysql
#设置mysql的安装目录
basedir=/usr/local/mysql
#mysql.sock存放目录
socket=/var/lib/mysql/mysql.sock
#设置mysql数据库的数据的存放目录
datadir=/usr/local/mysql/data
#日志文件输出
log-error=/var/log/mariadb/mariadb.log
#pid文件
pid-file=/var/run/mariadb/mariadb.pid
#设置创建数据库默认的字符集
character-set-server=utf8mb4
#默认存储引擎
default-storage-engine=innoDB
#查询语句等一系列操作将不区分大小写，会自动识别。
lower_case_table_names=1
#MySQL存放临时文件的目录
tmpdir = /data/tmpdate
```

扩展设置:

```shell
#允许最大连接数，默认100，最大16384
#根据需求来看，一般2核4G机器填写1000，16核64G填写5000。
max_connections = 5000
#当客户端连接延迟次数超过该值会报错
max_connect_errors = 6000
#建立三次握手的超时时间
connect_timeout=10
#控制连接最大空闲时长的参数。
#wait_timeout控制非交互，比如java程序的链接，interactive_timeout控制交互，比如mysql命令进行的操作。
#show global variables like '%timeout%';
wait_timeout = 300
interactive_timeout = 300
#如果等待连接的数量超过back_log，将不被授予连接资源
#不能超过TCP/IP连接的侦听队列的大小：cat /proc/sys/net/ipv4/tcp_max_syn_backlog
back_log = 300
#mysql可以打开的最大文件数，不能超过 ulimt -n 看到的数值
open_files_limit = 65535
#每当MySQL访问一个表时，如果在表缓冲区中还有空间，该表就被打开并放入其中，这样可以更快地访问表内容。
table_open_cache = 16000
#mysql根据配置文件会限制server接收的数据包大小。
max_allowed_packet = 500M

max_heap_table_size = 64M
tmp_table_size = 256M
#对表进行顺序扫描的请求将分配一个读入缓冲区
read_buffer_size = 8M

read_rnd_buffer_size = 8M
#系统中对数据进行排序的时候用到的Buffer
sort_buffer_size = 8M
#当使用join命令时，为了减少参与join的“被驱动表”的读取次数以提高性能，需要使用到join buffer来协助完成join操作
join_buffer_size = 8M
#指定索引缓冲区的大小，它决定索引处理的速度，尤其是索引读的速度。
#SHOW VARIABLES LIKE '%key_buffer_size%';
key_buffer_size = 256M
#每个客户端连接线程被创建时，MySQL给它分配的内存大小。
#show global status like 'Thread%';
thread_cache_size = 64
thread_stack = 512K
#如果没使用全文索引，就不要开启
ft_min_word_len = 1
#避免MySQL的外部锁定，减少出错几率增强稳定性。
skip-external-locking
bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 32M
#如果读或写一个通信端口中断，mysql放弃前尝试连接的次数。
net_retry_count = 100
#一般用在主主同步中，用来错开自增值, 防止键值冲突
auto_increment_increment=0
auto_increment_offset=0

explicit_defaults_for_timestamp=false
#如果开启了主从复制，要设置为0，禁止用户创建函数，触发器。因为存储函数有可能导致主从的数据不一致。
#如果只开启Binlog，没主从，则设置为1。
log_bin_trust_function_creators=1
#收集性能参数
performance_schema= 0
transaction-isolation = REPEATABLE-READ
query_cache_size = 0
query_cache_type = 0
```

binlog:

```shell
#开启binlog日志，mysql-bin 是日志的基本名或前缀名，可以更换
log-bin=mysql-bin
log-bin-index=mysql-bin.index
server-id=1
max_binlog_size = 512M
#binlog日志格式，mysql默认采用statement，建议使用mixed
binlog_format = MIXED
#把relay-log里的日志内容再记录到slave本地的binlog里，默认关闭，主主就要开启。
log_slave_updates = 0
#超过指定天数的binlog将被删除
expire_logs_days = 7
#从服务器在主服务器上复制的binlog日志，如果超过这个值，将形成一个新的日志。
max_relay_log_size = 512M
#binlog-ignore-db无需开启二进制日志文件的数据库，多个数据库则要重复设置
binlog-ignore-db = mysql
binlog-ignore-db = test
binlog-ignore-db = information_schema
binlog-ignore-db = performance_schema
#replicate-ignore-db来设置不需要同步的库
replicate-ignore-db = mysql
replicate-ignore-db = test
replicate-ignore-db = information_schema
replicate-ignore-db = performance_schema
#二进制日志缓冲区
binlog_cache_size = 20M
#超过这个数值，将会把日志写到一个新的文件中,最大值1GB
max_binlog_cache_size = 15M
```

慢查询:

```shell
#开启慢查询日志
#show variables like '%slow%';
slow_query_log=1
#指定超过多少秒返回查询的结果为慢查询，单位秒
long_query_time=2
#指定保存路径及文件名
slow_query_log_file=/data/hostname-slow.log
#记录所有没有使用到索引的查询语句，但可能会导致日志激增。
log-queries-not-using-indexes = TRUE
#表示每分钟允许记录到slow log的且未使用索引的sql语句次数，日志将不会激增
log_throttle_queries_not_using_indexes=1000
#记录那些由于查找了多余1000次而引发的慢查询
min_examined_row_limit=1000
log-slow-admin-statements = TRUE
```

innodb:

```shell
innodb_file_per_table = 1
#限制Innodb能打开的表的数据，默认为300
innodb_open_files = 1000
#这个是Innodb最重要的参数，主要作用是缓存innodb表的索引，数据，插入数据时的缓冲
innodb_buffer_pool_size = 48G

innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_flush_log_at_trx_commit = 1
#事务在内存中的缓冲，也就是日志缓冲区的大小，默认设置即可
innodb_log_buffer_size = 128M
#指定在一个日志组中，每个log的大小
#innodb的logfile就是事务日志，用来在mysql crash后的恢复。所以设置合理的大小对于mysql的性能非常重要，直接影响数据库的写入速度，事务大小，异常重启后的恢复。
innodb_log_file_size = 128M
#当系统中 脏页 所占百分比超过这个值，INNODB就会进行写操作以把页中的已更新数据写入到磁盘文件中
innodb_max_dirty_pages_pct = 85
innodb_lock_wait_timeout = 120
innodb_flush_method = O_DIRECT
innodb_data_file_path = ibdata1:10M:autoextend
innodb_autoinc_lock_mode  = 2
innodb_buffer_pool_dump_at_shutdown = 1
innodb_buffer_pool_load_at_startup  = 1
#设置为1，标志支持分布式事物，主要保证binary log和其他引擎的主事务数据保持一致性，属于同步操作；
#如果你设置0，就是异步操作，这样就会一定程度上减少磁盘的刷新次数和磁盘的竞争。
innodb_support_xa = 0
#SHOW INNODB STATUS 的输出每15秒钟写到一个状态文件。这个文件的名字是innodb_status.pid，其中pid是服务器进程ID。这个文件在MySQL数据目录里创建。
innodb_status_file  = 1
```



## 16c64g优化

以下配置适合16核64G及以上的配置，会让性能稍微提高1/3左右。

my.cnf

```shell
[client]
port = 3306
socket = /usr/local/mysql/mysql.sock
[mysqld]
#基础设置
port = 3306
bind-address = 0.0.0.0
lower_case_table_names=1
character-set-server=utf8mb4
default-storage-engine=innoDB
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/mysql.sock
log-error=/var/log/mysql/mysql.log
pid-file=/usr/local/mysql/mysql.pid

#扩展设置
max_connections = 5000
max_connect_errors = 6000
connect_timeout=10
wait_timeout = 300
interactive_timeout = 300
back_log = 300
open_files_limit = 65535
table_open_cache = 16000
max_allowed_packet = 500M
max_heap_table_size = 64M
tmp_table_size = 256M
read_buffer_size = 8M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 8M
key_buffer_size = 256M
thread_cache_size = 64
thread_stack = 512K
ft_min_word_len = 1
skip-external-locking
bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 32M
net_retry_count = 100
auto_increment_increment=0
auto_increment_offset=0
explicit_defaults_for_timestamp=false
log_bin_trust_function_creators=1
performance_schema= 0
transaction-isolation = REPEATABLE-READ
query_cache_size = 0
query_cache_type = 0

#binlog日志
log-bin=mysql-bin
log-bin-index=mysql-bin.index
server-id=1
max_binlog_size = 512M
binlog_format = MIXED
log_slave_updates = 0
expire_logs_days = 7
max_relay_log_size = 512M
binlog-ignore-db = mysql
binlog-ignore-db = test
binlog-ignore-db = information_schema
binlog-ignore-db = performance_schema
replicate-ignore-db = mysql
replicate-ignore-db = test
replicate-ignore-db = information_schema
replicate-ignore-db = performance_schema
binlog_cache_size = 20M
max_binlog_cache_size = 15M

#慢查询
slow_query_log=1
long_query_time=2
log-queries-not-using-indexes = TRUE
log_throttle_queries_not_using_indexes=1000
min_examined_row_limit=1000
log-slow-admin-statements = TRUE
log-slow-admin-statements = TRUE

#innodb引擎
innodb_file_per_table = 1
innodb_open_files = 1000
innodb_buffer_pool_size = 48G
innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_flush_log_at_trx_commit = 1
innodb_log_buffer_size = 128M
innodb_log_file_size = 128M
innodb_max_dirty_pages_pct = 85
innodb_lock_wait_timeout = 120
innodb_flush_method = O_DIRECT
innodb_data_file_path = ibdata1:10M:autoextend
innodb_autoinc_lock_mode  = 2
innodb_buffer_pool_dump_at_shutdown = 1
innodb_buffer_pool_load_at_startup  = 1
innodb_support_xa = 0
innodb_status_file  = 1
```

## 4c8g优化

以下配置适合4核8G及以下的配置，会让性能稍微提高1/3左右。

测试语句
`mysqlslap -uroot -p123456 --concurrency=100 --iterations=30 --auto-generate-sql --auto-generate-sql-load-type=mixed --auto-generate-sql-add-autoincrement --engine=innodb --number-of-queries=5000`

```
没配置
    Average number of seconds to run all queries: 0.735 seconds
    Minimum number of seconds to run all queries: 0.551 seconds
    Maximum number of seconds to run all queries: 1.141 seconds
优化后的
    Average number of seconds to run all queries: 0.691 seconds
    Minimum number of seconds to run all queries: 0.630 seconds
    Maximum number of seconds to run all queries: 0.749 seconds
```

my.cnf

```shell
[client]
port = 3306
socket = /usr/local/mysql/mysql.sock
[mysqld]
#基础设置
port = 3306
bind-address = 0.0.0.0
lower_case_table_names=1
character-set-server=utf8mb4
default-storage-engine=innoDB
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/mysql.sock
log-error=/var/log/mysql/mysql.log
pid-file=/usr/local/mysql/mysql.pid

#扩展设置
max_connections = 1000
max_connect_errors = 6000
connect_timeout=10
wait_timeout = 300
interactive_timeout = 300
back_log = 300
open_files_limit = 65535
table_open_cache = 512
max_allowed_packet = 500M
max_heap_table_size = 8M
tmp_table_size = 64M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 8M
key_buffer_size = 64M
thread_cache_size = 32
thread_stack = 128K
ft_min_word_len = 1
skip-external-locking
bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 32M
net_retry_count = 100
auto_increment_increment=0
auto_increment_offset=0
explicit_defaults_for_timestamp=false
log_bin_trust_function_creators=1
performance_schema= 0
transaction-isolation = REPEATABLE-READ
query_cache_size = 0
query_cache_type = 0

#binlog日志
log-bin=mysql-bin
log-bin-index=mysql-bin.index
server-id=1
max_binlog_size = 512M
binlog_format = MIXED
log_slave_updates = 0
expire_logs_days = 7
max_relay_log_size = 512M
binlog-ignore-db = mysql
binlog-ignore-db = test
binlog-ignore-db = information_schema
binlog-ignore-db = performance_schema
replicate-ignore-db = mysql
replicate-ignore-db = test
replicate-ignore-db = information_schema
replicate-ignore-db = performance_schema
binlog_cache_size = 1M
max_binlog_cache_size = 15M

#慢查询
slow_query_log=1
long_query_time=1
log-queries-not-using-indexes = TRUE
log_throttle_queries_not_using_indexes=1000
min_examined_row_limit=1000
log-slow-admin-statements = TRUE
log-slow-admin-statements = TRUE

#innodb引擎
innodb_file_per_table = 1
innodb_open_files = 500
innodb_buffer_pool_size = 512M
innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 32M
innodb_max_dirty_pages_pct = 85
innodb_lock_wait_timeout = 120
innodb_flush_method=O_DIRECT
innodb_data_file_path = ibdata1:10M:autoextend
innodb_autoinc_lock_mode  = 2
innodb_buffer_pool_dump_at_shutdown = 1
innodb_buffer_pool_load_at_startup  = 1
innodb_support_xa = 0
innodb_status_file  = 1
```

