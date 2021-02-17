# binlog

## 简介

binlog是msyql最重要的日志，它记录了ddl和dml语句（除了查询语句），以事件形式记录，还包含语句所执行的消耗和时间。MySQL的二进制日志是事务安全型的。一般来说开启binlog日志大概会有1%的性能损耗

DDL
主要的命令有CREATE、ALTER、DROP等，DDL主要是用在定义或改变表（TABLE）的结构，数据类型，表之间的链接和约束等初始化工作上，他们大多在建立表时使用。

DML
主要的命令是SELECT、UPDATE、INSERT、DELETE，就象它的名字一样，这4条命令是用来对数据库里的数据进行操作的语言

**使用场景**：

1. 主从复制
2. 备份恢复

binlog日志包括**两类文件**：
1）二进制日志索引文件（文件名后缀为.index）用于记录所有的二进制文件
2）二进制日志文件（文件名后缀为.00000*）记录数据库所有的DDL和DML(除了数据查询语句select)语句事件。

## 开启binlog和相关参数

### 开启

```shell
vim /etc/my.cnf
#开启，并且可以将mysql-bin改为其它的日志名
log-bin=mysql-bin
#添加id号，如果做主从，就不能一样
server-id=1
#超过200M将生产新的文件，最大和默认值是1GB
max_binlog_size=1G
#此参数配置binlog的日志格式，默认为mixed。
binlog_format=mixed
#此参数表示binlog使用最大内存的数。
max_binlog_cache_size=1M
#此参数表示只记录指定数据库的二进制日志。
binlog-do-db=数据库A
#此参数表示不记录指定的数据库的二进制日志。
binlog-ignore-db=数据库A
#此参数表示binlog日志保留的时间，默认单位是天。
expire_logs_days=7
重启服务
```

### 相关操作

查看所有binlog日志列表

```mysql
show master logs;
```

查看binlog日志是否开启

```mysql
show variables like 'log_%';
```

![image-20210206164842768](https://gitee.com/c_honghui/picture/raw/master/img/20210217233404.png)

查看master状态，即最后(最新)一个binlog日志的编号名称，及其最后一个操作事件pos结束点(Position)值

```mysql
show master status;
```

![image-20210206164950491](https://gitee.com/c_honghui/picture/raw/master/img/20210217233408.png)

flush刷新log日志，自此刻开始产生一个新编号的binlog日志文件。每当mysqld服务重启时，会自动执行此命令，刷新binlog日志

```mysql
flush logs;
```

清空binlog

```mysql
reset master;
```

### 查看binlog日志

#### 使用mysqlbinlog自带查看

```shell
mysqlbinlog mysql-bin.000002
```

解释
server id 1 ： 数据库主机的服务号；
end_log_pos 796： sql结束时的pos节点
thread_id=11： 线程号

#### mysql加载方式查询

`show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];`

```text
IN 'log_name' ：指定要查询的binlog文件名(不指定就是第一个binlog文件)
FROM pos ：指定从哪个pos起始点开始查起(不指定就是从整个文件首个pos点开始算)
LIMIT [offset,] ：偏移量(不指定就是0)
row_count ：查询总条数(不指定就是所有行)
```

其余例子
a）查询第一个(最早)的binlog日志：
`show binlog events\G;`

b）指定查询 mysql-bin.000002这个文件：
`show binlog events in 'mysql-bin.000002'\G;`

c）指定查询 mysql-bin.000002这个文件，从pos点:624开始查起：
`show binlog events in 'mysql-bin.000002' from 624\G;`

d）指定查询 mysql-bin.000002这个文件，从pos点:624开始查起，查询10条（即10条语句）
`show binlog events in 'mysql-bin.000002' from 624 limit 10\G;`

e）指定查询 mysql-bin.000002这个文件，从pos点:624开始查起，偏移2行（即中间跳过2个），查询10条
`show binlog events in 'mysql-bin.000002' from 624 limit 2,10\G;`

## 恢复数据

说明：
恢复的时候要配合全备份，先进行全备份，在用mysqldump全备时添加-F刷新binlog，这时候mysqldump备份的是最新的binlog日志之前的内容了。

先进行全备份恢复，再将最新的binlog文件用mysqlbinlog进行查看，grep或者其他方式过滤，找到有问题的sql语句，记录下当时的pos点或者时间。只恢复出问题之前得时间点即可。

如果只是某个数据库有问题，可以只恢复单个数据库而不是所有：
`mysqlbinlog mysql-bin.000015 –database=数据库A | mysql -uroot -p’123456‘ 数据库A`

参数：

```text
常用参数选项解释：
--start-position=875 起始pos点
--stop-position=954 结束pos点
--start-datetime="2016-9-25 22:01:08" 起始时间点
--stop-datetime="2019-9-25 22:09:46" 结束时间点
--database=zyyshop 指定只恢复zyyshop数据库(一台主机上往往有多个数据库，只限本地log日志)
```

