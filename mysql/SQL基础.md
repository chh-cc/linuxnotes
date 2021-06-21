# SQL基础

结构化查询语言

分类:

```undefined
DDL：数据定义语言
DCL：数据控制语言
DML：数据操作语言
DQL：数据的查询语言
```

## 数据类型

### 整型

![file](https://gitee.com/c_honghui/picture/raw/master/img/20210217233555.png)

```text
取值范围如果加了unsigned，则最大值翻倍，如tinyint unsigned的取值范围为(0~256)。
int(m)里的m是表示SELECT查询结果集中的显示宽度

手机号无法存入int，一般用char存储
```

### 浮点型

![file](https://gitee.com/c_honghui/picture/raw/master/img/20210217233604.png)

设一个字段定义为float(6,3)，如果插入一个数123.45678,实际数据库里存的是123.457，但总个数还以实际为准，即6位。整数部分最大是3位，如果插入数12.123456，存储的是12.1234，如果插入12.12，存储的是12.1200.

### 定点数

浮点型在数据库中存放的是近似值，而定点类型在数据库中存放的是精确值。

decimal(m,d) 参数m<65 是总个数，d<30且 d<m 是小数位。

### 字符串

![file](https://gitee.com/c_honghui/picture/raw/master/img/20210217233609.png)

char和varchar：

```text
1.char(11) ：定长的字符串类型,在存储字符串时，最大字符长度11个，立即分配11个字符长度的存储空间，如果存不满，空格填充。

2.varchar(11): 变长的字符串类型，最大字符长度11个。在存储字符串时，自动判断字符长度，按需分配存储空间。

3.char类型的字符串检索速度要比varchar类型的快。
```



![img](https://gitee.com/c_honghui/picture/raw/master/img/20210217233614.png)

### 二进制数据

1.BLOB和text存储方式不同，TEXT以文本方式存储，英文存储区分大小写，而Blob是以二进制方式存储，不分大小写。

2.BLOB存储的数据只能整体读出。

3.TEXT可以指定字符集，BLO不用指定字符集。

### 时间日期类型

![file](https://gitee.com/c_honghui/picture/raw/master/img/20210217233624.png)

若定义一个字段为timestamp，这个字段里的时间数据会随其他字段修改的时候自动刷新，所以这个数据类型的字段可以存放这条记录最后被修改的时间。

## 属性

### 列属性

约束（一般建表时添加）：

| 主键（primary key ） | 唯一、非空 ，主键在一个表只能有一个，数字列、整数列、无关列、自增 |
| -------------------- | ------------------------------------------------------------ |
| 非空（not null）     | 普通列尽量设置not null，可以设置默认为0                      |
| 唯一（unique）       | 不能重复                                                     |
| 无符号（unsigned ）  | 无符号，针对数字列非负数                                     |

其他属性：

| 索引           | 在某列上建立索引来优化查询，一般是根据需要后添加             |
| -------------- | ------------------------------------------------------------ |
| default        | 默认值，列中没有录入值时自动使用default值                    |
| auto_increment | 自增，针对数字列，顺序的自动填充数据，默认从1开始，可以设定起始和偏移量 |
| comment        | 注释                                                         |

### 表属性

```undefined
存储引擎:
InnoDB（默认的）
字符集和排序规则:
utf8       
utf8mb4
```

## DDL(数据库定义语言)

库

```mysql
#创建数据库
create database school;
create schema sch;

show charset;
show collation;
CREATE DATABASE test CHARSET utf8;
create database xyz charset utf8mb4 collate utf8mb4_bin;

建库规范：
1.库名不能有大写字母   
2.建库要加字符集         
3.库名不能有数字开头
4.库名要和业务相关

#删库(生产禁止)
mysql> drop database oldboy;

#修改
alter database db_name default charset utf8;                         #修改数据库的字符集 
注意：修改字符集，修改后的字符集一定是原字符集的严格超集
```

表

```mysql
#创建表
create table stu(
列1  属性（数据类型、约束、其他属性），
列2  属性，
列3  属性
)

USE school;
CREATE TABLE stu(
id      INT NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT '学号',
sname   VARCHAR(255) NOT NULL COMMENT '姓名',
sage    TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '年龄',
sgender ENUM('m','f','n') NOT NULL DEFAULT 'n' COMMENT '性别',
sfz     CHAR(18) NOT NULL UNIQUE  COMMENT '身份证',
intime  TIMESTAMP NOT NULL DEFAULT NOW() COMMENT '入学时间'
) ENGINE=INNODB CHARSET=utf8 COMMENT '学生表';

建表规范：
1. 表名小写
2. 不能是数字开头
3. 注意字符集和存储引擎
4. 表名和业务有关
5. 选择合适的数据类型
6. 每个列都要有注释
7. 每个列设置为非空，无法保证非空，用0来填充。

将s1复制为s2表
create table s2 select * from s1;
复制s1的表结构为s2，不包含数据
create table IF NOT EXISTS s2 (LIKE s1);

#删表（生产禁止）
drop table t1;

#修改
1.在stu表中添加qq列
DESC stu;
ALTER TABLE stu ADD qq VARCHAR(20) NOT NULL UNIQUE COMMENT 'qq号';
2.在sname后加微信列
ALTER TABLE stu ADD wechat VARCHAR(64) NOT NULL UNIQUE  COMMENT '微信号' AFTER sname;
3.在id列前加一个新列num
ALTER TABLE stu ADD num INT NOT NULL COMMENT '数字' FIRST;
DESC stu;
4.把刚才添加的列都删掉(危险)
ALTER TABLE stu DROP num;
ALTER TABLE stu DROP qq;
ALTER TABLE stu DROP wechat;
5.修改sname数据类型的属性
ALTER TABLE stu MODIFY sname VARCHAR(128)  NOT NULL;
6.将sgender 改为 sg 数据类型改为 CHAR 类型
ALTER TABLE stu CHANGE sgender sg CHAR(1) NOT NULL DEFAULT 'n' ;
DESC stu;
7.将表s1名字改成s2
alter table s1 rename to s2;
8.更改默认存储引擎
alter table s2 ENGINE=InnoDB;

alter table tab_name default charset utf8 collate utf8_general_ci;   #修改表字符集和字符序
```

## DQL(数据库查询语言)

库

```mysql
show databases；
show create database database_name \G                                #查看数据库字符集设置
```

表

```mysql
use school #打开数据库
show tables；#查看库中所有的表
desc stu; #查看表结构
show create table table_name \G                                      #查看表字符集设置

show table status             # 查看表的引擎状态

explain select * from s1; #查询过程中的操作信息
```

单独使用

```mysql
-- select @@xxx 查看系统参数
SELECT @@port;
SELECT @@basedir;
SELECT @@datadir;
SELECT @@socket;
SELECT @@server_id;

SELECT NOW();
SELECT DATABASE();
SELECT USER();
select version();
SELECT CONCAT("hello world");
SELECT CONCAT(USER,"@",HOST) FROM mysql.user;
SELECT GROUP_CONCAT(USER,"@",HOST) FROM mysql.user;
```

其他

```mysql
show processlist;             # 查看mysql进程
show full processlist;        # 显示进程全的语句
show slave status\G;          # 查看主从状态
show variables;               # 查看所有参数变量
show status;                  # 运行状态
show grants for user@'%';     # 查看用户权限
show OPEN TABLES where In_use > 0;                  # 查看当前锁信息
show variables like 'innodb_print_all_deadlocks';   # 查看当前死锁参数
show global variables like '%char%';                #查看RDS实例字符集相关参数设置
show global variables like 'coll%';                 #查看当前会话字符序相关参数设置
show character set;                                 #查看实例支持的字符集
show collation;                                     #查看实例支持的字符序
```

单表子句-from

```mysql
SELECT 列1,列2 FROM 表
SELECT  *  FROM 表

SELECT * FROM stu ; #查询stu中所有的数据(不要对大表进行操作)
select user,host from mysql.user; #显示所有用户
```

单表子句-where

```mysql
SELECT col1,col2 FROM TABLE WHERE colN 条件;

SELECT * FROM city WHERE countrycode='CHN'; #查询中国(CHN)所有城市信息

where配合比较操作符(> < >= <= <>)
SELECT * FROM city WHERE population<100; #查询世界上少于100人的城市

where配合逻辑运算符(and or )
SELECT * FROM city WHERE countrycode='CHN' AND population>5000000; #中国人口数量大于500w

where配合模糊查询
SELECT * FROM city WHERE district LIKE 'guang%'; #查询省的名字前面带guang开头的，注意:%不能放在前面,因为不走索引.

where配合in语句
SELECT * FROM city WHERE countrycode IN ('CHN' ,'USA'); #中国或美国城市信息

where配合between and
SELECT * FROM city  WHERE population BETWEEN 1000000 AND 2000000; #查询世界上人口数量大于100w小于200w的城市信息
```

group by+常用聚合函数

```mysql
根据 by后面的条件进行分组，方便统计，by后面跟一个列或多个列

常用聚合函数
**max()**      ：最大值
**min()**      ：最小值
**avg()**      ：平均值
**sum()**      ：总和
**count()**    ：个数
group_concat() : 列转行

统计世界上每个国家的总人口数
USE world;
SELECT countrycode,SUM(population) FROM city GROUP BY countrycode;

统计中国各个省的总人口数量(练习)
SELECT district,SUM(Population) FROM city  WHERE countrycode='chn' GROUP BY district;
```

having

```mysql
统计中国每个省的总人口数，只打印总人口数小于100
SELECT district,SUM(Population)
FROM city
WHERE countrycode='chn'
GROUP BY district
HAVING SUM(Population) < 1000000 ;
```

order by+limit

```mysql
实现先排序，by后添加条件列

查看中国所有的城市，并按人口数进行排序(从大到小)
SELECT * FROM city WHERE countrycode='CHN' ORDER BY population DESC;

统计中国各个省的总人口数量，按照总人口从大到小排序
SELECT district AS 省 ,SUM(Population) AS 总人口
FROM city
WHERE countrycode='chn'
GROUP BY district
ORDER BY 总人口 DESC ;

统计中国,每个省的总人口,找出总人口大于500w的,并按总人口从大到小排序,只显示前三名
SELECT  district, SUM(population)  FROM  city 
WHERE countrycode='CHN'
GROUP BY district 
HAVING SUM(population)>5000000
ORDER BY SUM(population) DESC
LIMIT 3 ;

LIMIT N ,M --->跳过N,显示一共M行
LIMIT 5,5

SELECT  district, SUM(population)  FROM  city 
WHERE countrycode='CHN'
GROUP BY district 
HAVING SUM(population)>5000000
ORDER BY SUM(population) DESC
LIMIT 5,5;
```

distinct

```mysql
去重复
SELECT countrycode FROM city ;
SELECT DISTINCT(countrycode) FROM city  ;
```

union all

```mysql
联合查询
-- 中国或美国城市信息

SELECT * FROM city 
WHERE countrycode IN ('CHN' ,'USA');

SELECT * FROM city WHERE countrycode='CHN'
UNION ALL
SELECT * FROM city WHERE countrycode='USA'

说明:一般情况下,我们会将 IN 或者 OR 语句 改写成 UNION ALL,来提高性能
UNION     去重复
UNION ALL 不去重复
```

join

```mysql
多表连接查询

案例准备：
use school
student ：学生表
sno：    学号
sname：学生姓名
sage： 学生年龄
ssex： 学生性别

teacher ：教师表
tno：     教师编号
tname：教师名字

course ：课程表
cno：  课程编号
cname：课程名字
tno：  教师编号

score  ：成绩表
sno：  学号
cno：  课程编号
score：成绩

-- 项目构建
drop database school;
CREATE DATABASE school CHARSET utf8;
USE school

CREATE TABLE student(
sno INT NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT '学号',
sname VARCHAR(20) NOT NULL COMMENT '姓名',
sage TINYINT UNSIGNED  NOT NULL COMMENT '年龄',
ssex  ENUM('f','m') NOT NULL DEFAULT 'm' COMMENT '性别'
)ENGINE=INNODB CHARSET=utf8;

CREATE TABLE course(
cno INT NOT NULL PRIMARY KEY COMMENT '课程编号',
cname VARCHAR(20) NOT NULL COMMENT '课程名字',
tno INT NOT NULL  COMMENT '教师编号'
)ENGINE=INNODB CHARSET utf8;

CREATE TABLE sc (
sno INT NOT NULL COMMENT '学号',
cno INT NOT NULL COMMENT '课程编号',
score INT  NOT NULL DEFAULT 0 COMMENT '成绩'
)ENGINE=INNODB CHARSET=utf8;

CREATE TABLE teacher(
tno INT NOT NULL PRIMARY KEY COMMENT '教师编号',
tname VARCHAR(20) NOT NULL COMMENT '教师名字'
)ENGINE=INNODB CHARSET utf8;

INSERT INTO student(sno,sname,sage,ssex)
VALUES (1,'zhang3',18,'m');

INSERT INTO student(sno,sname,sage,ssex)
VALUES
(2,'zhang4',18,'m'),
(3,'li4',18,'m'),
(4,'wang5',19,'f');

INSERT INTO student
VALUES
(5,'zh4',18,'m'),
(6,'zhao4',18,'m'),
(7,'ma6',19,'f');

INSERT INTO student(sname,sage,ssex)
VALUES
('oldboy',20,'m'),
('oldgirl',20,'f'),
('oldp',25,'m');


INSERT INTO teacher(tno,tname) VALUES
(101,'oldboy'),
(102,'hesw'),
(103,'oldguo');

DESC course;
INSERT INTO course(cno,cname,tno)
VALUES
(1001,'linux',101),
(1002,'python',102),
(1003,'mysql',103);

DESC sc;
INSERT INTO sc(sno,cno,score)
VALUES
(1,1001,80),
(1,1002,59),
(2,1002,90),
(2,1003,100),
(3,1001,99),
(3,1003,40),
(4,1001,79),
(4,1002,61),
(4,1003,99),
(5,1003,40),
(6,1001,89),
(6,1003,77),
(7,1001,67),
(7,1003,82),
(8,1001,70),
(9,1003,80),
(10,1003,96);

SELECT * FROM student;
SELECT * FROM teacher;
SELECT * FROM course;
SELECT * FROM sc;

查询张三的家庭住址
SELECT A.name,B.address FROM
A JOIN  B
ON A.id=B.id
WHERE A.name='zhangsan'

查询一下世界上人口数量小于100人的城市名和国家名
SELECT b.name ,a.name ,a.population
FROM  city  AS a
JOIN  country AS b
ON    b.code=a.countrycode
WHERE  a.Population<100

查询城市shenyang，城市人口，所在国家名（name）及国土面积（SurfaceArea）
SELECT a.name,a.population,b.name ,b.SurfaceArea
FROM city  AS a JOIN country AS b
ON a.countrycode=b.code
WHERE a.name='shenyang';
```



## DCL（数据库控制语言）

```mysql
#授权
grant 权限列表 ON 库名.表名 TO 用户名@'客户端地址';
grant all on zabbix.* to user@"$IP";             # 对现有账号赋予权限
grant select on database.* to user@"%" Identified by "passwd";     # 赋予查询权限(没有用户，直接创建，不允许对当前库给其他用户赋权限)
grant all privileges on database.* to user@"localhost" identified by 'passwd' with grant option;   # 赋予本机指定用户所有权限(允许对当前库给其他用户赋权限)
grant select, insert, update, delete on database.* to user@'ip'identified by "passwd";   # 开放管理操作指令

#撤销权限
revoke all on *.* from user@localhost;     # 回收权限
```

## DML（数据库操作语言）

对表中的数据行进行增、删、改

insert

```mysql
--- 最标准的insert语句
INSERT INTO stu(id,sname,sage,sg,sfz,intime) 
VALUES
(1,'zs',18,'m','123456',NOW());
SELECT * FROM stu;
--- 省事的写法
INSERT INTO stu 
VALUES
(2,'ls',18,'m','1234567',NOW());
--- 针对性的录入数据
INSERT INTO stu(sname,sfz)
VALUES ('w5','34445788');
--- 同时录入多行数据
INSERT INTO stu(sname,sfz)
VALUES 
('w55','3444578d8'),
('m6','1212313'),
('aa','123213123123');
SELECT * FROM stu;
```

update

```mysql
DESC stu;
SELECT * FROM stu;
UPDATE stu SET sname='zhao4' WHERE id=2;
注意：update语句必须要加where。
```

delete（危险）

```mysql
DELETE FROM stu  WHERE id=3;

全表删除:
DELETE FROM stu
truncate table stu;
区别:
delete: DML操作, 是逻辑性质删除,逐行进行删除,速度慢.
truncate: DDL操作,对与表段中的数据页进行清空,速度快.

伪删除：用update来替代delete，最终保证业务中查不到（select）即可
1.添加状态列
ALTER TABLE stu ADD state TINYINT NOT NULL DEFAULT 1 ;
SELECT * FROM stu;
2. UPDATE 替代 DELETE
UPDATE stu SET state=0 WHERE id=6;
3. 业务语句查询
SELECT * FROM stu WHERE state=1;
```