# 分布式架构mycat

架构图：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210704214008.webp)

环境准备：

```undefined
两台虚拟机 db01 db02
每台创建四个mysql实例：3307 3308 3309 3310
```

创建相关目录初始化数据

```shell
mkdir /data/33{07..10}/data -p
mysqld --initialize-insecure  --user=mysql --datadir=/data/3307/data --basedir=/app/mysql
mysqld --initialize-insecure  --user=mysql --datadir=/data/3308/data --basedir=/app/mysql
mysqld --initialize-insecure  --user=mysql --datadir=/data/3309/data --basedir=/app/mysql
mysqld --initialize-insecure  --user=mysql --datadir=/data/3310/data --basedir=/app/mysql
```

准备配置文件和启动脚本

```jsx
========db01==============
cat >/data/3307/my.cnf<<EOF
[mysqld]
basedir=/app/mysql
datadir=/data/3307/data
socket=/data/3307/mysql.sock
port=3307
log-error=/data/3307/mysql.log
log_bin=/data/3307/mysql-bin
binlog_format=row
skip-name-resolve
server-id=7
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
EOF

cat >/data/3308/my.cnf<<EOF
[mysqld]
basedir=/app/mysql
datadir=/data/3308/data
port=3308
socket=/data/3308/mysql.sock
log-error=/data/3308/mysql.log
log_bin=/data/3308/mysql-bin
binlog_format=row
skip-name-resolve
server-id=8
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
EOF

cat >/data/3309/my.cnf<<EOF
[mysqld]
basedir=/app/mysql
datadir=/data/3309/data
socket=/data/3309/mysql.sock
port=3309
log-error=/data/3309/mysql.log
log_bin=/data/3309/mysql-bin
binlog_format=row
skip-name-resolve
server-id=9
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
EOF
cat >/data/3310/my.cnf<<EOF
[mysqld]
basedir=/app/mysql
datadir=/data/3310/data
socket=/data/3310/mysql.sock
port=3310
log-error=/data/3310/mysql.log
log_bin=/data/3310/mysql-bin
binlog_format=row
skip-name-resolve
server-id=10
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
EOF

cat >/etc/systemd/system/mysqld3307.service<<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/app/mysql/bin/mysqld --defaults-file=/data/3307/my.cnf
LimitNOFILE = 5000
EOF

cat >/etc/systemd/system/mysqld3308.service<<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/app/mysql/bin/mysqld --defaults-file=/data/3308/my.cnf
LimitNOFILE = 5000
EOF

cat >/etc/systemd/system/mysqld3309.service<<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/app/mysql/bin/mysqld --defaults-file=/data/3309/my.cnf
LimitNOFILE = 5000
EOF
cat >/etc/systemd/system/mysqld3310.service<<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/app/mysql/bin/mysqld --defaults-file=/data/3310/my.cnf
LimitNOFILE = 5000
EOF
========db02===============
cat >/data/3307/my.cnf<<EOF
[mysqld]
basedir=/app/mysql
datadir=/data/3307/data
socket=/data/3307/mysql.sock
port=3307
log-error=/data/3307/mysql.log
log_bin=/data/3307/mysql-bin
binlog_format=row
skip-name-resolve
server-id=17
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
EOF
cat >/data/3308/my.cnf<<EOF
[mysqld]
basedir=/app/mysql
datadir=/data/3308/data
port=3308
socket=/data/3308/mysql.sock
log-error=/data/3308/mysql.log
log_bin=/data/3308/mysql-bin
binlog_format=row
skip-name-resolve
server-id=18
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
EOF
cat >/data/3309/my.cnf<<EOF
[mysqld]
basedir=/app/mysql
datadir=/data/3309/data
socket=/data/3309/mysql.sock
port=3309
log-error=/data/3309/mysql.log
log_bin=/data/3309/mysql-bin
binlog_format=row
skip-name-resolve
server-id=19
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
EOF


cat >/data/3310/my.cnf<<EOF
[mysqld]
basedir=/app/mysql
datadir=/data/3310/data
socket=/data/3310/mysql.sock
port=3310
log-error=/data/3310/mysql.log
log_bin=/data/3310/mysql-bin
binlog_format=row
skip-name-resolve
server-id=20
gtid-mode=on
enforce-gtid-consistency=true
log-slave-updates=1
EOF

cat >/etc/systemd/system/mysqld3307.service<<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/app/mysql/bin/mysqld --defaults-file=/data/3307/my.cnf
LimitNOFILE = 5000
EOF

cat >/etc/systemd/system/mysqld3308.service<<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/app/mysql/bin/mysqld --defaults-file=/data/3308/my.cnf
LimitNOFILE = 5000
EOF

cat >/etc/systemd/system/mysqld3309.service<<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/app/mysql/bin/mysqld --defaults-file=/data/3309/my.cnf
LimitNOFILE = 5000
EOF
cat >/etc/systemd/system/mysqld3310.service<<EOF
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/app/mysql/bin/mysqld --defaults-file=/data/3310/my.cnf
LimitNOFILE = 5000
EOF
```

修改权限，启动多实例

```shell
chown -R mysql.mysql /data/*
systemctl start mysqld3307
systemctl start mysqld3308
systemctl start mysqld3309
systemctl start mysqld3310

mysql -S /data/3307/mysql.sock -e "show variables like 'server_id'"
mysql -S /data/3308/mysql.sock -e "show variables like 'server_id'"
mysql -S /data/3309/mysql.sock -e "show variables like 'server_id'"
mysql -S /data/3310/mysql.sock -e "show variables like 'server_id'"
```

节点主从规划

```css
箭头指向谁是主库
    10.0.0.51:3307    <----->  10.0.0.52:3307
    10.0.0.51:3309    ------>  10.0.0.51:3307
    10.0.0.52:3309    ------>  10.0.0.52:3307

    10.0.0.52:3308  <----->    10.0.0.51:3308
    10.0.0.52:3310  ----->     10.0.0.52:3308
    10.0.0.51:3310  ----->     10.0.0.51:3308
```

分片规划

```css
shard1：
    Master：10.0.0.51:3307
    slave1：10.0.0.51:3309
    Standby Master：10.0.0.52:3307
    slave2：10.0.0.52:3309
shard2：
    Master：10.0.0.52:3308
    slave1：10.0.0.52:3310
    Standby Master：10.0.0.51:3308
    slave2：10.0.0.51:3310
```

配置：

shard1

10.0.0.51:3307 <-----> 10.0.0.52:3307

```shell
db02：
mysql  -S /data/3307/mysql.sock -e "grant replication slave on *.* to repl@'10.0.0.%' identified by '123';"
mysql  -S /data/3307/mysql.sock -e "grant all  on *.* to root@'10.0.0.%' identified by '123'  with grant option;"

db01：
mysql  -S /data/3307/mysql.sock -e "CHANGE MASTER TO MASTER_HOST='10.0.0.52', MASTER_PORT=3307, MASTER_AUTO_POSITION=1, MASTER_USER='repl', MASTER_PASSWORD='123';"
mysql  -S /data/3307/mysql.sock -e "start slave;"
mysql  -S /data/3307/mysql.sock -e "show slave status\G"

db02：
mysql  -S /data/3307/mysql.sock -e "CHANGE MASTER TO MASTER_HOST='10.0.0.51', MASTER_PORT=3307, MASTER_AUTO_POSITION=1, MASTER_USER='repl', MASTER_PASSWORD='123';"
mysql  -S /data/3307/mysql.sock -e "start slave;"
mysql  -S /data/3307/mysql.sock -e "show slave status\G"
```

10.0.0.51:3309 ------> 10.0.0.51:3307

```shell
db01：
mysql  -S /data/3309/mysql.sock  -e "CHANGE MASTER TO MASTER_HOST='10.0.0.51', MASTER_PORT=3307, MASTER_AUTO_POSITION=1, MASTER_USER='repl', MASTER_PASSWORD='123';"
mysql  -S /data/3309/mysql.sock  -e "start slave;"
mysql  -S /data/3309/mysql.sock  -e "show slave status\G"
```

10.0.0.52:3309 ------> 10.0.0.52:3307

```kotlin
db02：
mysql  -S /data/3309/mysql.sock -e "CHANGE MASTER TO MASTER_HOST='10.0.0.52', MASTER_PORT=3307, MASTER_AUTO_POSITION=1, MASTER_USER='repl', MASTER_PASSWORD='123';"
mysql  -S /data/3309/mysql.sock -e "start slave;"
mysql  -S /data/3309/mysql.sock -e "show slave status\G"
```

shard2

10.0.0.52:3308 <-----> 10.0.0.51:3308

```kotlin
db01：
mysql  -S /data/3308/mysql.sock -e "grant replication slave on *.* to repl@'10.0.0.%' identified by '123';"
mysql  -S /data/3308/mysql.sock -e "grant all  on *.* to root@'10.0.0.%' identified by '123'  with grant option;"

db02：
mysql  -S /data/3308/mysql.sock -e "CHANGE MASTER TO MASTER_HOST='10.0.0.51', MASTER_PORT=3308, MASTER_AUTO_POSITION=1, MASTER_USER='repl', MASTER_PASSWORD='123';"
mysql  -S /data/3308/mysql.sock -e "start slave;"
mysql  -S /data/3308/mysql.sock -e "show slave status\G"

db01：
mysql  -S /data/3308/mysql.sock -e "CHANGE MASTER TO MASTER_HOST='10.0.0.52', MASTER_PORT=3308, MASTER_AUTO_POSITION=1, MASTER_USER='repl', MASTER_PASSWORD='123';"
mysql  -S /data/3308/mysql.sock -e "start slave;"
mysql  -S /data/3308/mysql.sock -e "show slave status\G"
```

10.0.0.52:3310 -----> 10.0.0.52:3308

```kotlin
db02：
mysql  -S /data/3310/mysql.sock -e "CHANGE MASTER TO MASTER_HOST='10.0.0.52', MASTER_PORT=3308, MASTER_AUTO_POSITION=1, MASTER_USER='repl', MASTER_PASSWORD='123';"
mysql  -S /data/3310/mysql.sock -e "start slave;"
mysql  -S /data/3310/mysql.sock -e "show slave status\G"
```

10.0.0.51:3310 -----> 10.0.0.51:3308

```kotlin
db01：
mysql  -S /data/3310/mysql.sock -e "CHANGE MASTER TO MASTER_HOST='10.0.0.51', MASTER_PORT=3308, MASTER_AUTO_POSITION=1, MASTER_USER='repl', MASTER_PASSWORD='123';"
mysql  -S /data/3310/mysql.sock -e "start slave;"
mysql  -S /data/3310/mysql.sock -e "show slave status\G"
```

检测主从状态

```shell
mysql -S /data/3307/mysql.sock -e "show slave status\G"|grep Yes
mysql -S /data/3308/mysql.sock -e "show slave status\G"|grep Yes
mysql -S /data/3309/mysql.sock -e "show slave status\G"|grep Yes
mysql -S /data/3310/mysql.sock -e "show slave status\G"|grep Yes
注：如果中间出现错误，在每个节点进行执行以下命令
mysql -S /data/3307/mysql.sock -e "stop slave; reset slave all;"
mysql -S /data/3308/mysql.sock -e "stop slave; reset slave all;"
mysql -S /data/3309/mysql.sock -e "stop slave; reset slave all;"
mysql -S /data/3310/mysql.sock -e "stop slave; reset slave all;"
```

mycat安装：

```shell
yum install -y java

tar xf Mycat-server-1.6.5-release-20180122220033-linux.tar.gz

ls
bin  catlet  conf  lib  logs  version.txt

配置环境变量
vim /etc/profile
export PATH=/application/mycat/bin:$PATH
source /etc/profile
启动
mycat start
连接mycat：
mysql -uroot -p123456 -h 127.0.0.1 -P8066
```

用户创建及数据库导入

```kotlin
db01:
mysql -S /data/3307/mysql.sock 
grant all on *.* to root@'10.0.0.%' identified by '123';
source /root/world.sql

mysql -S /data/3308/mysql.sock 
grant all on *.* to root@'10.0.0.%' identified by '123';
source /root/world.sql
```

配置文件处理

```xml
cd /application/mycat/conf
mv schema.xml schema.xml.bak
vim schema.xml 
<?xml version="1.0"?>  
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">  
<mycat:schema xmlns:mycat="http://io.mycat/">
<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1"> 
</schema>  
    <dataNode name="dn1" dataHost="localhost1" database= "wordpress" />  
    <dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"  writeType="0" dbType="mysql"  dbDriver="native" switchType="1"> 
        <heartbeat>select user()</heartbeat>  
    <writeHost host="db1" url="10.0.0.51:3307" user="root" password="123"> 
            <readHost host="db2" url="10.0.0.51:3309" user="root" password="123" /> 
    </writeHost> 
    </dataHost>  
</mycat:schema>
```

读写分离结构配置

```xml
vim schema.xml 

<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">  
<mycat:schema xmlns:mycat="http://io.mycat/">
<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="sh1"> 
</schema>  
        <dataNode name="sh1" dataHost="oldguo1" database= "world" />         
        <dataHost name="oldguo1" maxCon="1000" minCon="10" balance="1"  writeType="0" dbType="mysql"  dbDriver="native" switchType="1">    
                <heartbeat>select user()</heartbeat>  
        <writeHost host="db1" url="10.0.0.51:3307" user="root" password="123"> 
                        <readHost host="db2" url="10.0.0.51:3309" user="root" password="123" /> 
        </writeHost> 
        </dataHost>  
</mycat:schema>

重启mycat
mycat restart

读写分离测试
 mysql -uroot -p -h 127.0.0.1 -P8066
 show variables like 'server_id';
 begin;
 show variables like 'server_id';

总结： 
以上案例实现了1主1从的读写分离功能，写操作落到主库，读操作落到从库.如果主库宕机，从库不能在继续提供服务了。
```

配置读写分离及高可用

```xml
[root@db01 conf]# mv schema.xml schema.xml.rw
[root@db01 conf]# vim schema.xml

<?xml version="1.0"?>  
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">  
<mycat:schema xmlns:mycat="http://io.mycat/">
<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="sh1"> 
</schema>  
    <dataNode name="sh1" dataHost="oldguo1" database= "world" />  
    <dataHost name="oldguo1" maxCon="1000" minCon="10" balance="1"  writeType="0" dbType="mysql"  dbDriver="native" switchType="1"> 
        <heartbeat>select user()</heartbeat>  
    <writeHost host="db1" url="10.0.0.51:3307" user="root" password="123"> 
            <readHost host="db2" url="10.0.0.51:3309" user="root" password="123" /> 
    </writeHost> 
    <writeHost host="db3" url="10.0.0.52:3307" user="root" password="123"> 
            <readHost host="db4" url="10.0.0.52:3309" user="root" password="123" /> 
    </writeHost>        
    </dataHost>  
</mycat:schema>

真正的 writehost：负责写操作的writehost  
standby  writeHost  ：和readhost一样，只提供读服务

当写节点宕机后，后面跟的readhost也不提供服务，这时候standby的writehost就提供写服务，
后面跟的readhost提供读服务

测试：
mysql -uroot -p123456 -h 127.0.0.1 -P 8066
show variables like 'server_id';
读写分离测试
 mysql -uroot -p -h 127.0.0.1 -P8066
 show variables like 'server_id';
 show variables like 'server_id';
 show variables like 'server_id';
 begin;
 show variables like 'server_id';
 对db01 3307节点进行关闭和启动,测试读写操作
 
```