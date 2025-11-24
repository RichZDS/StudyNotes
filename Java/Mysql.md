# 先前的

mysql -u root -p

123456

## sql

其实分为四大语言

DDL 定义

DML 管理

DCL 控制

DQL 查询

## 进入各种库

```
user 库名   进入库
show tables;  显示库中的表名
describe 表名;  显示表中的所有数据类型
create databases 库名字  建立数据库
```



## 数据恢复

SOURCE 数据库地址

注意 要在mysql条件下运行

## 数据保存

mysqldump -u root -p -B 数据库名 > 地址

这是存库的

mysqldump -u root -p 

注意该指令要在cmd环境下运行 不在进入mysql中

## 创建库

```mysql
CREATE TABLE `user3` ( id INT, `name` VARCHAR ( 255 ), `address` VARCHAR ( 255 ), `birthday` DATE );
这是没有
CREATE TABLE `user3`(
id INT,
`name` varchar(255),
`address` varchar (255),
`birthday` DATE
)
CHARACTER SET utf8 COLLATE utf8_bin ENGINE INNODB;
这是完整的
```

## 操作数据库

1.建立数据库

```
create database dbName
```

2.删除数据库

```
drop database dbName
```

3使用数据库

```
use dbName
```

4.查看数据库

```
show databases ------看所有的
```

## 数据库的字段属性

- Unsigned 无符号整数 声明了该列不能为负数
- zerofill 零填充
- 自增（auto_increment） 自动在上一条的代码上面+1 通常用来设置唯一的主键
- 非空(not null) 不能不设值 不填就是nunll
- 默认(dafault) 设置默认的值
- 注释(comment)

## 阿里巴巴规范手则

```
id   主键
`version`   乐观锁
is_delete  	伪删除
gmt_create 创建时间
gmt_update 修改时间
```



## 一般格式

```mysql
CREATE TABLE IF NOT EXISIT `表名`(
  `字段名` 列类型 属性 索引 注释,
  `字段名` 列类型 属性 索引 注释,
    *****
   `字段名` 列类型 属性 索引 注释
)


CREATE TABLE `user1`(
`id` INT(4) NOT NULL COMMENT '账号id',
`name` VARCHAR(10) DEFAULT '无名' ,
`pwd` VARCHAR(10) NOT NULL ,
`sex` VARCHAR(7) DEFAULT'沃尔玛购物袋' COMMENT '性别',
PRIMARY KEY (`id`)
)DEFAULT CHARSET=utf8
```

## 逆向代码

```mysql
SHOW CREATE DATABASE 库名 --查看数据库怎么创建的语句
SHOW CREATE TABLE tableName
DESC table Name
```

## 修改

```mysql
修改表名
ALTER TABLE `user` RENAME AS `useralert`;
增加表中的属性
ALTER TABLE `useralert` ADD `age` INT(4);
改属性 **MODIFY**
ALTER TABLE `useralert` MODIFY `age` VARCHAR(20)
改名字 **MODIFY**
ALTER TABLE `useralert` MODIFY `age` `agealert` INT(1);
删属性
ALTER TABLE `useralert` DROP `agealert`;
删除表
 DROP TABLE tableName
```



## 外键 fk_

```mysql
KEY `FK_name` (`name`),
constraint `FK_name` FOREGIN KEY (`name`) references tablename (`reference_name`)
就像是把数值绑定在一起 引用

****一般不这么写了

****一般这么写 但是也不怎么用
ALTER TABLE tableName ADD CONSTRAINT  `FK_name` FOREGIN KEY (`name`) references tablename (`reference_name`)
```

删除有外键关系的表 必须要把引用表删掉

所以现在基本都不怎么用外键 数据库级别的外键 不怎么用 删起来太TM麻烦了

==最佳实践==

DB就是单纯的表 只是用来存数据 行（数据）列（字段）

以后用多张表的数据 就通过程序实现

## DML语言（全部记住）

### 添加

```MYSQL
INSERT INTO tableName (字段名1，字段名2, .....) values (数据1,数据2，......)

**自适应**
INSERT INTO user1 VALUES('12','小白','3599','01');
--一次插入多行数据--
INSERT INTO user1 (name,pwd,sex) VALUES ("小白","3599","0"),("小","3599","0");

主键 自适应 可以不写
```

巨大的错误

![image-20241024223149529](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241024223149529.png)

在创建表的时候 如果是这么写的 那么mysql的后面会自动帮你添加一个``

![image-20241024223241503](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241024223241503.png)

但是 如果你又加了一个`` 

![image-20241024223318611](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241024223318611.png)

那么其实mysql给你建立的是

![image-20241024223337832](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241024223337832.png)

所以当你引用

```
`id`  不要问为什么使用代码块 因为在md格式中 “ ``  ”是有语法的
```

的时候 就会报错

1054 - Unknown column 'name' in 'field list'

哈哈哈

其实你应该写 

```
```id```
```

哈哈哈哈哈哈 太tm操蛋了 所以家人们 不要瞎几把用图形界面

![](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241102205149319.png)

### 修改

```mysql
UPDATE tableName 字段=新值 ，如果多个属性就用,   WHERE 字段=要找的 --也可以是要修改的 

WHERE 还可以用> < BETWEEN... AND... 闭合区间  单独用AND OR
```

### 删除

```mysql
DELETE FROM tableName WHERE 条件 

- 如果表引擎为INNODB  自增列会从1重新开始 （存在内存当中的，断电即失） - 如果表引擎为MyISAM   会自动从上一个自增量开始 （存在文件中的，不会丢失）

TRUNCATE tablename 删除表
TRUNCATE 不会影响事务 并且 自增列 计数器归零


```

## DQL (data query language)查询数据

```mysql
SELECT 东西 [AS `别名`] FROM tablename
SELECT CONCAT()函数 类似拼接
SELECT CONCAT ('名字：',studentname) AS 新名字 FROM student;

SELECT DISTINCT studentno FROM result; 去重查询

SELECT 3*4 AS 计算结果 hhh好玩

SELECT studentresult,subjectno FROM result WHERE studentresult>60&&studentresult<90;加了一点点限定WHERE AND



LIKE语法 像 %（0-任何数字）   _一个字符
SELECT studentname FROM student WHERE studentname LIKE '张_'  ;
SELECT studentname FROM student WHERE studentname LIKE '张__'  ;
SELECT studentname FROM student WHERE studentname LIKE '张%'  ;

SELECT studentname FROM student WHERE studentname LIKE '%白%'  ;查找名字有白的人

===IN(精确查找)===
SELECT studentname FROM student WHERE address IN ('北京朝阳');
IN （）内可以加，来获得更多的 
```

## 七种查询方式

![image-20241027002535996](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241027002535996.png)

### 联表查询

```mysql
INNER JOIN 交集
SELECT
  s.studentno,
  studentname 
FROM
  student AS s
  INNER JOIN result AS r 
WHERE
  s.studentno = r.studentno
  LEFT JOIN 返回左表所有的值和右表匹配的值
```

![s](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241102205211125.png)

SELECT语句的顺序

## 分页LIMIT

```
LIMIT 起始值，页大小
```

## GROUP BY

```mysql
SELECT subjectname,AVG(studentresult),MAX(studentresult),MIN(studentresult)
FROM result r
INNER JOIN `subject` sub
ON r.subjectno=sub.subjectno
GROUP BY r.subjectno
=========================================================
1：只要有聚合函数 sum()，count()，max()，avg() 等函数就需要用到 group by , 否则就会报上面的错误.
2：group by id (id 是主键) 的时候, select 什么都没有问题, 包括有聚合函数.
3：group by role (非主键) 的时候, select 只能是聚合函数和 role ( group by 的字段) , 否则报错
```

## 事务

 

```mysql
SET autocommit =0  --关闭自动提交
SET autocommit =0 --开启自动提交

START TRANSACTION ---标记一个事务的开始 如果事务中的一个sql语句插入失败 那么整条事务都不会插入
 
COMMIT --提交
ROLLBACK --回滚
SAVEPOINT 存档名 --中间暂存
ROLLBACK TO 存档名
RELEASE SAVEPOINT 存档名 --删除
```

## 索引

#### 主键索引

#### 唯一索引

避免列重复出现

#### 常规索引

默认的 index key 关键字来设置

#### 全文索引

### 索引原则

索引不是越多越好

不要对经常变动的数据加索引 查询加

小数据 5百万



## 狂神说MySQL06：事务和索引



## 事务

> 什么是事务

- 事务就是将一组SQL语句放在同一批次内去执行
- 如果一个SQL语句出错,则该批次内的所有SQL都将被取消执行
- MySQL事务处理只支持InnoDB和BDB数据表类型

> 事务的ACID原则  百度 ACID

**原子性(Atomic)**

- 整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回滚（ROLLBACK）到事务开始前的状态，就像这个事务从来没有执行过一样。

**一致性(Consist)**

- 一个事务可以封装状态改变（除非它是一个只读的）。事务必须始终保持系统处于一致的状态，不管在任何给定的时间并发事务有多少。也就是说：如果事务是并发多个，系统也必须如同串行事务一样操作。其主要特征是保护性和不变性(Preserving an Invariant)，以转账案例为例，假设有五个账户，每个账户余额是100元，那么五个账户总额是500元，如果在这个5个账户之间同时发生多个转账，无论并发多少个，比如在A与B账户之间转账5元，在C与D账户之间转账10元，在B与E之间转账15元，五个账户总额也应该还是500元，这就是保护性和不变性。

**隔离性(Isolated)**

- 隔离状态执行事务，使它们好像是系统在给定时间内执行的唯一操作。如果有两个事务，运行在相同的时间内，执行相同的功能，事务的隔离性将确保每一事务在系统中认为只有该事务在使用系统。这种属性有时称为串行化，为了防止事务操作间的混淆，必须串行化或序列化请求，使得在同一时间仅有一个请求用于同一数据。

**持久性(Durable)**

- 在事务完成以后，该事务对数据库所作的更改便持久的保存在数据库之中，并不会被回滚。

> **基本语法**

```sql
-- 使用set语句来改变自动提交模式
SET autocommit = 0;   /*关闭*/
SET autocommit = 1;   /*开启*/

-- 注意:
--- 1.MySQL中默认是自动提交
--- 2.使用事务时应先关闭自动提交

-- 开始一个事务,标记事务的起始点
START TRANSACTION  

-- 提交一个事务给数据库
COMMIT

-- 将事务回滚,数据回到本次事务的初始状态
ROLLBACK

-- 还原MySQL数据库的自动提交
SET autocommit =1;

-- 保存点
SAVEPOINT 保存点名称 -- 设置一个事务保存点
ROLLBACK TO SAVEPOINT 保存点名称 -- 回滚到保存点
RELEASE SAVEPOINT 保存点名称 -- 删除保存点
```



> 测试

```sql
/*
课堂测试题目

A在线买一款价格为500元商品,网上银行转账.
A的银行卡余额为2000,然后给商家B支付500.
商家B一开始的银行卡余额为10000

创建数据库shop和创建表account并插入2条数据
*/

CREATE DATABASE `shop`CHARACTER SET utf8 COLLATE utf8_general_ci;
USE `shop`;

CREATE TABLE `account` (
`id` INT(11) NOT NULL AUTO_INCREMENT,
`name` VARCHAR(32) NOT NULL,
`cash` DECIMAL(9,2) NOT NULL,
PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO account (`name`,`cash`)
VALUES('A',2000.00),('B',10000.00)

-- 转账实现
SET autocommit = 0; -- 关闭自动提交
START TRANSACTION;  -- 开始一个事务,标记事务的起始点
UPDATE account SET cash=cash-500 WHERE `name`='A';
UPDATE account SET cash=cash+500 WHERE `name`='B';
COMMIT; -- 提交事务
# rollback;
SET autocommit = 1; -- 恢复自动提交
```



## 索引

> 索引的作用

- 提高查询速度
- 确保数据的唯一性
- 可以加速表和表之间的连接 , 实现表与表之间的参照完整性
- 使用分组和排序子句进行数据检索时 , 可以显著减少分组和排序的时间
- 全文检索字段进行搜索优化.

> 分类

- 主键索引 (Primary Key)
- 唯一索引 (Unique)
- 常规索引 (Index)
- 全文索引 (FullText)

> 主键索引

主键 : 某一个属性组能唯一标识一条记录

特点 :

- 最常见的索引类型
- 确保数据记录的唯一性
- 确定特定数据记录在数据库中的位置

> 唯一索引

作用 : 避免同一个表中某数据列中的值重复

与主键索引的区别

- 主键索引只能有一个
- 唯一索引可能有多个

```sql
CREATE TABLE `Grade`(
  `GradeID` INT(11) AUTO_INCREMENT PRIMARYKEY,
  `GradeName` VARCHAR(32) NOT NULL UNIQUE
   -- 或 UNIQUE KEY `GradeID` (`GradeID`)
)
```

> 常规索引

作用 : 快速定位特定数据

注意 :

- index 和 key 关键字都可以设置常规索引
- 应加在查询找条件的字段
- 不宜添加太多常规索引,影响数据的插入,删除和修改操作

```sql
CREATE TABLE `result`(
   -- 省略一些代码
  INDEX/KEY `ind` (`studentNo`,`subjectNo`) -- 创建表时添加
)
-- 创建后添加
ALTER TABLE `result` ADD INDEX `ind`(`studentNo`,`subjectNo`);
```

> 全文索引

百度搜索：全文索引

作用 : 快速定位特定数据

注意 :

- 只能用于MyISAM类型的数据表
- 只能用于CHAR , VARCHAR , TEXT数据列类型
- 适合大型数据集

```sql
/*
#方法一：创建表时
  　　CREATE TABLE 表名 (
               字段名1 数据类型 [完整性约束条件…],
               字段名2 数据类型 [完整性约束条件…],
               [UNIQUE | FULLTEXT | SPATIAL ]   INDEX | KEY
               [索引名] (字段名[(长度)] [ASC |DESC])
               );


#方法二：CREATE在已存在的表上创建索引
       CREATE [UNIQUE | FULLTEXT | SPATIAL ] INDEX 索引名
                    ON 表名 (字段名[(长度)] [ASC |DESC]) ;


#方法三：ALTER TABLE在已存在的表上创建索引
       ALTER TABLE 表名 ADD [UNIQUE | FULLTEXT | SPATIAL ] INDEX
                            索引名 (字段名[(长度)] [ASC |DESC]) ;
                           
                           
#删除索引：DROP INDEX 索引名 ON 表名字;
#删除主键索引: ALTER TABLE 表名 DROP PRIMARY KEY;


#显示索引信息: SHOW INDEX FROM student;
*/

/*增加全文索引*/
ALTER TABLE `school`.`student` ADD FULLTEXT INDEX `studentname` (`StudentName`);

/*EXPLAIN : 分析SQL语句执行性能*/
EXPLAIN SELECT * FROM student WHERE studentno='1000';

/*使用全文索引*/
-- 全文搜索通过 MATCH() 函数完成。
-- 搜索字符串作为 against() 的参数被给定。搜索以忽略字母大小写的方式执行。对于表中的每个记录行，MATCH() 返回一个相关性值。即，在搜索字符串与记录行在 MATCH() 列表中指定的列的文本之间的相似性尺度。
EXPLAIN SELECT *FROM student WHERE MATCH(studentname) AGAINST('love');

/*
开始之前，先说一下全文索引的版本、存储引擎、数据类型的支持情况

MySQL 5.6 以前的版本，只有 MyISAM 存储引擎支持全文索引；
MySQL 5.6 及以后的版本，MyISAM 和 InnoDB 存储引擎均支持全文索引;
只有字段的数据类型为 char、varchar、text 及其系列才可以建全文索引。
测试或使用全文索引时，要先看一下自己的 MySQL 版本、存储引擎和数据类型是否支持全文索引。
*/
```

> 拓展：测试索引

**建表app_user：**

```sql
CREATE TABLE `app_user` (
`id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
`name` varchar(50) DEFAULT '' COMMENT '用户昵称',
`email` varchar(50) NOT NULL COMMENT '用户邮箱',
`phone` varchar(20) DEFAULT '' COMMENT '手机号',
`gender` tinyint(4) unsigned DEFAULT '0' COMMENT '性别（0:男；1：女）',
`password` varchar(100) NOT NULL COMMENT '密码',
`age` tinyint(4) DEFAULT '0' COMMENT '年龄',
`create_time` datetime DEFAULT CURRENT_TIMESTAMP,
`update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='app用户表'
```

**批量插入数据：100w**

```sql
DROP FUNCTION IF EXISTS mock_data;
DELIMITER $$
CREATE FUNCTION mock_data()
RETURNS INT
BEGIN
DECLARE num INT DEFAULT 1000000;
DECLARE i INT DEFAULT 0;
WHILE i < num DO
  INSERT INTO app_user(`name`, `email`, `phone`, `gender`, `password`, `age`)
   VALUES(CONCAT('用户', i), '24736743@qq.com', CONCAT('18', FLOOR(RAND()*(999999999-100000000)+100000000)),FLOOR(RAND()*2),UUID(), FLOOR(RAND()*100));
  SET i = i + 1;
END WHILE;
RETURN i;
END;
SELECT mock_data();
```

**索引效率测试**

无索引

```sql
SELECT * FROM app_user WHERE name = '用户9999'; -- 查看耗时
SELECT * FROM app_user WHERE name = '用户9999';
SELECT * FROM app_user WHERE name = '用户9999';

mysql> EXPLAIN SELECT * FROM app_user WHERE name = '用户9999'\G
*************************** 1. row ***************************
          id: 1
select_type: SIMPLE
       table: app_user
  partitions: NULL
        type: ALL
possible_keys: NULL
        key: NULL
    key_len: NULL
        ref: NULL
        rows: 992759
    filtered: 10.00
      Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

创建索引

```sql
CREATE INDEX idx_app_user_name ON app_user(name);
```

测试普通索引

```sql
mysql> EXPLAIN SELECT * FROM app_user WHERE name = '用户9999'\G
*************************** 1. row ***************************
          id: 1
select_type: SIMPLE
       table: app_user
  partitions: NULL
        type: ref
possible_keys: idx_app_user_name
        key: idx_app_user_name
    key_len: 203
        ref: const
        rows: 1
    filtered: 100.00
      Extra: NULL
1 row in set, 1 warning (0.00 sec)

mysql> SELECT * FROM app_user WHERE name = '用户9999';
1 row in set (0.00 sec)

mysql> SELECT * FROM app_user WHERE name = '用户9999';
1 row in set (0.00 sec)

mysql> SELECT * FROM app_user WHERE name = '用户9999';
1 row in set (0.00 sec)
```

> 索引准则

- 索引不是越多越好
- 不要对经常变动的数据加索引
- 小数据量的表建议不要加索引
- 索引一般应加在查找条件的字段

> 索引的数据结构

```sql
-- 我们可以在创建上述索引的时候，为其指定索引类型，分两类
hash类型的索引：查询单条快，范围查询慢
btree类型的索引：b+树，层数越多，数据量指数级增长（我们就用它，因为innodb默认支持它）

-- 不同的存储引擎支持的索引类型也不一样
InnoDB 支持事务，支持行级别锁定，支持 B-tree、Full-text 等索引，不支持 Hash 索引；
MyISAM 不支持事务，支持表级别锁定，支持 B-tree、Full-text 等索引，不支持 Hash 索引；
Memory 不支持事务，支持表级别锁定，支持 B-tree、Hash 等索引，不支持 Full-text 索引；
NDB 支持事务，支持行级别锁定，支持 Hash 索引，不支持 B-tree、Full-text 等索引；
Archive 不支持事务，支持表级别锁定，不支持 B-tree、Hash、Full-
text 等索引；
```

## ![image-20241119123819985](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241119123819985.png)







# 大三新一轮学习

## distinct无法部分使用

## limit

limit 3，4 从第三行开始数四个 等价于 limit 4 offset 3

## concat

SELECT **CONCAT(prod_name,' 的价格为 ',prod_price)** FROM products 



## Trim

RTrim 删除右边的空白 LTrim删除左边的空白 Trim()去掉串左右两边的空格

## LOW_priority

```mysql
insert low_priority into (update delete同样适用) 这样会降低语句的优先级然后提高其他用户的查询
```





# 大三数据库

## exists

查询中的一种特殊类型是 "exists" 子查询，用于检查主查询的结果集是否存在满足条件的记录，它返回布尔值（True 或 False），而不返回实际的数据。



```sql
-- 主查询
SELECT name, total_amount
FROM customers
WHERE EXISTS (
    -- 子查询
    SELECT 1
    FROM orders
    WHERE orders.customer_id = customers.customer_id
);
```

 



## union

###  在 SQL 中，组合查询是一种将多个 SELECT 查询结果合并在一起的查询操作。

>包括两种常见的组合查询操作：UNION 和 UNION ALL。

1. UNION 操作：它用于将两个或多个查询的结果集合并， **并去除重复的行** 。即如果两个查询的结果有相同的行，则只保留一行。
2. UNION ALL 操作：它也用于将两个或多个查询的结果集合并， **但不去除重复的行** 。即如果两个查询的结果有相同的行，则全部保留。



## 开窗函数 - sum over

在 SQL 中，开窗函数是一种强大的查询工具，它允许我们在查询中进行对分组数据进行计算、 **同时保留原始行的详细信息** 。

开窗函数可以与聚合函数（如 SUM、AVG、COUNT 等）结合使用，

> 但与普通聚合函数不同，开窗函数不会导致结果集的行数减少。

打个比方，可以将开窗函数想象成一种 "透视镜"，它能够将我们聚焦在某个特定的分组，同时还能看到整体的全景。



## sum over order by

> sum over order by，可以实现同组内数据的 **累加求和** 。

```sql
SUM(计算字段名) OVER (PARTITION BY 分组字段名 ORDER BY 排序字段 排序规则)
```



##  rank

简明知意

```sql
RANK() OVER (
  PARTITION BY 列名1, 列名2, ... -- 可选，用于指定分组列
  ORDER BY 列名3 [ASC|DESC], 列名4 [ASC|DESC], ... -- 用于指定排序列及排序方式
) AS rank_column
```





### 触发器 

当xxx的时候 就会发生的一个特殊的 你懂的



## 角色管理

```sql
-- 完成数据库新用户创建、
CREATE USER `zds`
-- 授权访问数据库、
-- 基本语法就是 grant 操作 on table 表名字 to 用户、
GRANT  SELECT, UPDATE, INSERT ON TABLE sales TO zds;

-- 授权用户传递权限（with grant option）
GRANT SELECT ON sdut.sales TO zds WITH GRANT OPTION
-- 收回用户的sales表更新权限
REVOKE UPDATE ON sdut.sales FROM zds;

-- 收回所有权限（彻底回收）
REVOKE ALL PRIVILEGES ON sdut.sales FROM zds
--  删除创建的用户
DROP USER zds;

```



## 触发器

```sql
CREATE trigger 触发器名字 BEFORE|AFTER 触发事件     
ON 触发事件的操作表名 FOR EACH ROW （sql语句）表示激活触发器后被执行的语句
```

