# mysql语句

## 用户

### 显示用户

授权库mysql主要的几个表：

user表：存储授权用户的访问权限

db表：授权用户对数据库的访问权限

tables_priv表：授权用户对表的访问权限

columns_priv表：授权用户对字段的访问权限



显示所有用户（root权限）：

``` mysql
select user,host,password from mysql.user;
```

显示不重复用户：

``` mysql
select distinct user from mysql.user;
```

### 创建用户

客户端地址：

%：匹配所有主机

192.168.0.%：匹配某个网段

192.168.0.1：匹配单个ip

%.chen.com：匹配一个dns区域

blog.chen.com：匹配指定域名

create user 用户名@'客户端地址' identified by ‘密码’;



删除用户：

DROP USER 用户名@'客户端地址';

delete from mysql.user where user=’用户名’ and host=’客户端地址’

### 授权

```mysql
grant 权限列表 ON 库名.表名 TO 用户名@'客户端地址';
```

权限列表：

all #所有权限
select,update(字段1，字段2)

库名.表名
*.* #所有库和所有表



只读账号：

` grant select on *.* to username@'ip';`

撤销权限：

```mysql
revoke 权限列表 ON 库名.表名 用户名@'客户端地址';
```

查询某个用户的权限：

``` mysql
show grants for 用户名@'客户端地址';
```

## 密码

### 设置规则

这个其实与validate_password_policy的值有关，默认为1，所以刚开始设置的密码必须符合长度，且必须含有数字，小写或大写字母，特殊字符。
如果我们不希望密码设置的那么复杂，需要修改两个全局参数：`validate_password_length`默认值为8,最小值为4

`set global validate_password_policy=0;` 只验证长度
`set global validate_password_length=4；` 修改密码默认长度

### 用SET PASSWORD命令

配置root密码
`SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');`

用户修改自己密码
`SET PASSWORD=PASSWORD('newpass');`

### 用mysqladmin

```shell
mysqladmin -u root password "newpass"
```

如果root已经设置过密码，采用如下方法
`mysqladmin -u root password oldpass "newpass"`

### 更改当前用户密码

```msyql
ALTER USER USER() IDENTIFIED BY '123456';
```

### 用UPDATE直接编辑user表

```mysql
use mysql;`
`UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root';`
`FLUSH PRIVILEGES;
```

### root密码丢失

关闭验证密码
`mysqld_safe --skip-grant-tables&`

登陆
`mysql -u root mysql`

重置
`UPDATE user SET password=PASSWORD("new password") WHERE user='root';`

5.7版本
`UPDATE user SET authentication_string=PASSWORD("new password") WHERE user='root';`

刷新
`FLUSH PRIVILEGES;`



#### 查

查看现在时间

`select now()`

查看警告

`show warnings;`

输出a1表查询过程中的操作信息

`explain select * from s1;`

*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: a1
         type: ALL
possible_keys: NULL #没有用索引
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 6 #总共查询6行
        Extra:

查询建表语句：

`show create table s2;`

查看表结构：

`desc s1;`

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210217233844.png)

查看s1表的结构，显示name字段
`SHOW COLUMNS FROM s21 LIKE '%name';`



information_schema.tables视图

```text
DESC information_schema.TABLES;
TABLE_SCHEMA    ---->库名
TABLE_NAME      ---->表名
ENGINE          ---->引擎
TABLE_ROWS      ---->表的行数
AVG_ROW_LENGTH  ---->表中行的平均行（字节）
INDEX_LENGTH    ---->索引的占用空间大小（字节）
```

查询整个数据库中所有库和所对应的表信息:

```mysql
SELECT table_schema,GROUP_CONCAT(table_name)
FROM  information_schema.tables
GROUP BY table_schema;
```

查询所有innodb引擎的表及所在的库:

```mysql
SELECT table_schema,table_name,ENGINE FROM information_schema.`TABLES`
WHERE ENGINE='innodb';
```

统计world数据库下每张表的磁盘空间占用:

```mysql
SELECT table_name,CONCAT((TABLE_ROWS*AVG_ROW_LENGTH+INDEX_LENGTH)/1024," KB")  AS size_KB
FROM information_schema.tables WHERE TABLE_SCHEMA='world';
```

统计所有数据库的总的磁盘空间占用:

```mysql
SELECT
TABLE_SCHEMA,
CONCAT(SUM(TABLE_ROWS*AVG_ROW_LENGTH+INDEX_LENGTH)/1024," KB") AS Total_KB
FROM information_schema.tables
GROUP BY table_schema;
```

### 数据

#### 增

插入一条数据：

````mysql
desc s1;
insert into s1(name,age) values('xx',11),('dd',35);
````

#### 删

删除表所有记录
`DELETE FROM 表名;`

删除某个表id为3百万的记录，只要匹配就删除。
`delete from s1 where id=3000000;`

清空表数据-1
`truncate table table_name;`

清空表数据-2
`delete * from table_name;`
注 : truncate操作中的table可以省略，delete操作中的*可以省略

truncate、delete 清空表数据的区别 :

1. truncate 是整体删除 (速度较快)，delete是逐条删除 (速度较慢)
2. truncate 不写服务器 log，delete 写服务器 log，也就是 truncate 效率比 delete高的原因
3. truncate 不激活trigger (触发器)，但是会重置Identity (标识列、自增字段)，相当于自增列会被置为初始值，又重新从1开始记录，而不是接着原来的 ID数。而 delete 删除以后，identity 依旧是接着被删除的最近的那一条记录ID加1后进行记录。如果只需删除表中的部分记录，只能使用 DELETE语句配合 where条件
4. truncate操作中的table可以省略，delete操作中的*可以省略

#### 

where(> < >= <= <> and or):

`select * from world.city where countrycode='CHN';`

`select * from world.city where countrycode like 'C%';`(不要出现%在前面的情况，效率低)

`select * from world.city where population<10000 and population>20000;`



group by+聚合函数

常用聚合函数：

``` text
max()      ：最大值
min()      ：最小值
avg()      ：平均值
sum()      ：总和
count()    ：个数
group_concat() : 列转行
```

统计**每个国家**的总人口:

```mysql
select countrycode,sum(population) from world.city group by countrycode;
```

统计每个国家的城市个数:

```mysql
select countrycode,count(id) from world.city group by countrycode;
```

统计并显示每个国家的省名字并列出来:

```mysql
select countrycode,group_concat(district) from world.city group by countrycode;
```

统计中国每个省的城市列表:

``` mysql
select district,group_concat(name) from world.city where countrycode='CHN' group by district;
```



having

统计中国每个省的总人口数，只打印总人口数小于100：

`SELECT district,SUM(Population) FROM city WHERE countrycode='chn' GROUP BY district HAVING SUM(Population) < 1000000 ;`



order by + limit

查看中国所有的城市，并按人口数进行排序(从大到小):

`SELECT * FROM city WHERE countrycode='CHN' ORDER BY population DESC;`

统计中国各个省的总人口数量，按照总人口从大到小排序:

`SELECT district AS 省 ,SUM(Population) AS 总人口 FROM city WHERE countrycode='chn' GROUP BY district ORDER BY 总人口 DESC ;`

统计中国,每个省的总人口,找出总人口大于500w的,并按总人口从大到小排序,只显示前三名:

``` mysql
SELECT  district, SUM(population)  FROM  city 

WHERE countrycode='CHN'

GROUP BY district 

HAVING SUM(population)>5000000

ORDER BY SUM(population) DESC

LIMIT 3 ;
```



distinct去重复

```mysql
SELECT DISTINCT(countrycode) FROM city  ;
```



union all联合查询

中国或美国城市信息:

```mysql
SELECT * FROM city WHERE countrycode='CHN'
UNION ALL
SELECT * FROM city WHERE countrycode='USA'
UNION     去重复 
UNION ALL 不去重复
```

#### 改

修改某行数据的qq字段内容：

`update s1 set qq='123423424' where id='5';`