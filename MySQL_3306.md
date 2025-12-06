[toc]

## 一、安装 / 配置

### 1、使用 YUM 存储库安装 MySQL

> YUM 是 RedHat、CentOS、Oracle Linux 版本的软件存储库。
>
> [MySQL 安装教程](https://dev.mysql.com/doc/refman/5.7/en/linux-installation-yum-repo.html)

- 安装 MySQL 源

```
yum localinstall -y http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
```

- 安装 MySQL 

```
yum install mysql-community-server
```

### 2、使用 APT 存储库安装 MySQL

> APT 是 Debian 和 Ubuntu 版本的软件存储库。
>
> [MySQL 安装教程](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/)

### 3、配置

- 启动 MySQL  服务

```
systemctl start mysqld
```

- 获取初始密码

```
> grep 'temporary password' /var/log/mysqld.log
A temporary password is generated for root@localhost: Jqqskhz1Wr
```

- 连接 MySQL 服务器

```
> mysql -h127.0.0.1 -P3306 -uroot -p
Enter password:
```

- 修改密码

```
> SET PASSWORD = PASSWORD('123456//ZZZjjj'); 
# 密码必须复杂 需包含大小写特殊符号，否则无法修改成功
```

**Navicat 连接数据库**

① SSH 通道连接

② 通过主机连接

- 开放远程连接

```
use mysql;
update user set host = '%' where user = 'root';
```

- 刷新权限，使权限立即生效

```
flush privileges;
```

- 开放端口

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```



## 二、数据库的创建和使用（DDL）

### 1、数据库的基本操作

（1）查看所有数据库

```mysql
SHOW DATABASES;
```

（2）查看指定数据库

```mysql
SHOW DATABASES LIKE '%test%';
```

（3）创建数据库

```mysql
SHOW DATABASE 数据库名 CHARSET utf8;
```

（4）选择数据库

```mysql
USE 数据库名;
```

（5）修改数据库

```mysql
ALTER DATABASE 数据库名 CHARSET utf8;
```

（6）删除数据库

```mysql
DROP DATABASE 数据库名;
```

### 2、数据表的基本操作

（1）查看选中数据库中的所有表

```mysql
SHOW TABLES;
```

（2）创建数据表

```mysql
CREATE TABLE 表名(
    字段名 类型 [字段属性],
    字段名 类型 [字段属性],
    字段名 类型 [字段属性],
    ...
)[表选项];

# 字段属性（六大约束）
	NOT NULL	非空
	DEFAULT		默认值
	PRIMARY KEY 主键（可以在字段后增加关键字，也可以在表选项中用 primary key(字段)）
	 - AUTO_INCREMENT	自动增长
	UNIQUE KEY	唯一键（唯一键在非空的情况下不允许重复，可以有多个NULL）
	CHECK		检查（MySQL不支持）
	FOREIGN KEY 外键
	COMMENT 	注释
	 
# 表选项：
	ENGIN：存储引擎（MyISAM、InnoDB）
	CHARSET：字符集（utf8）
	COLLATE：校对集（utf8_general_ci）
	COMMENT：注释
	
注：
（1）查看系统字符集处理变量
	SHOW VARIABLES LIKE 'character_set%';
（2）查看自增长初始变量
	SHOW VARIABLES LIKE 'auto_increment_%';
	auto_increment_increment	步长
	auto_increment_offset		初始值
（3）外键约束
	外键只能使用innodb存储引擎。
	外键所在表（从表）不能插入主表中不存在的数据。
	主表也不能随意删除一个被从表引入的数据。
	外键约束主要约束的对象是主表的操作，从表只是不能插入主表中不存在的数据。
	约束模式：
		RESTRICT	严格模式（默认），不允许操作。
		CASCADE		级联模式，从表数据跟着主表数据一起变化。
		SET NULL	主表数据删除，从表数据置空（从表中对应的外键字段允许为空）。
		NO ACTION	不采取行动
	
```

（3）查看表结构（三种语法的结果一样）

```mysql
DESC 表名;
DESCRIBE 表名;
SHOW COLUMNS FROM 表名;
```

（4）查看建语表句（DDL）

```mysql
SHOW CREATE TABLE 表名;
```

（5）复制已有表的结构

```mysql
CREATE TABLE 表名 LIKE 数据库名.表名;
```

（6）修改表名

```mysql
RENAME TABLE 旧名 TO 新名;
```

（7）修改表选项

```mysql
ALTER TABLE 表名 表选项 值;
```

（8）增加字段

```mysql
ALTER TABLE 表名 ADD [COLUMN] 字段名 字段类型 [列属性] [位置 FIRST|AFTER 字段名];

FIRST：第一个位置
FIRST：某个字段之后（默认最后）
```

（9）修改字段名

```mysql
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 字段类型 [列属性]  [新位置];
```

（10）修改字段属性

```mysql
ALTER TABLE 表名 MODIFY 字段名 新类型 [新属性]  [新位置];
```

（11）删除字段

```mysql
ALTER TABLE 表名 DROP 字段名;
```

（12）删除表

```mysql
DROP TABLE 表名;
```



## 三、数据类型

### 1、数值类型

（1）整形

```mysql
tinyint		迷你整形，1字节
smallint	小整型，2字节
mediumint	中整型，3字节
int			标准整型，4字节
bigint		大整型，8字节

注：
	默认有符号，无符号类型需添加关键字 unsigned。
	显示长度：int(3) 代表数据是否可以达到指定的长度，但不会自动填充到最大长度，可以用 zerofill 实现左填充 0 达到最大长度，一旦使用 zerofill 就代表该字段为 unsigned。
```

（2）定点型（保证数据的精确）

```mysql
decimal(M,D)
	M：代表总长度，最大值不能超过65
	D：代表小数部分长度，最大不能超过30
	默认为（10，0）

注：
	浮点型的小数部分超过范围，会四舍五入，而由小数进位导致的整数部分超出系统可以接受，直接插入超过范围的数时，系统会报错。
	定点数的小数部分超过范围时也会四舍五入，如果小数进位导致整数超过范围，系统会报错。
```

（3）浮点型

```mysql
float	单精度，4字节（大概能保证7位左右的精度）
double	双精度，8字节（15位左右的精度）

语法：
	float	不指定小数位
	float(M,D)指定位数
```

### 2、日期和时间类型

```mysql
date	格式：YYYY-mm-dd，4字节
time	格式：HH:ii:ss，4字节
datetime	合并上面格式
timestemp	时间戳类型，不能为空，有默认值CURRENT_TIMESTEMP，数据更新时，更新为当前时间。4字节，最大2038年
```

### 3、字符串类型

```mysql
	char(n)		定长字符，n的长度：0-255
	varchar(n)	变长字符，n的长度：0-65535
	tinytext	1字节，实际存储字符数为：2^8+1
	text		2字节，实际存储字符数为：2^16+1
	mediumtext	3字节，实际存储字符数为：2^24+1
	longtext	4字节，实际存储字符数为：2^32+1
注：
	不用刻意选择text类型，系统会根据存储数据长度自动选择合适文本类型。
	如果数据长度超过255个字符，推荐使用text类型，而不是varchar类型。
```

### 4、枚举类型（单选）

```mysql
语法：
	enum(值1, 值2, ...)
	
注：枚举类型实际存储的是数据的索引下标。
意义：规范数据，节省空间。
```

### 5、集合类型（多选）

```mysql
语法：
	set(值1, 值2, ...)
	
	集合是一种多个数据选项可以同时保存的数据类型，本质是用二进制位存储每个数据项，1表示选中，0表示未选中。
	系统会自动计算存储单元。
	1字节，set()里面只能有8个选项
	2字节，set()里面只能有16个选项
	……
	存储在数据库中的形式：将二进制反转后转换成十进制
注：
	set 与 enum 一样不区分大小写。
```

## 四、函数和运算符

### 1、算数运算符

（1）算数运算符

```
+		只作加法运算符
-		减、负
*
/
%、MOD	模运算 
```

（2）数学函数

```
ABS(X)			绝对值
CEIL(X)			向上取整
FLOOR(X)		向下取整
MOD(N,M)		n%m 求余
PI()			π
POW(X,Y)		x的y次幂
SQRT(X)			求平方根
RAND()			获取 [0,1) 的随机数
ROUND(X,D)		四舍五入，保留指定小数位（默认小数位 0）
SIGN(X)			返回参数的符号（-1、0、1）
TRUNCATE(X,D)	截取到指定小数位，不取舍
```

### 2、比较运算符

（1）比较运算符

```
# 
>
>=
<
<=
=
<>、!=
<=>			安全等于，在比较 NULL 时，返回 0或1 而不是 NULL
```

（2）比较函数

```
BETWEEN x AND y		查找在该范围内的值
NOT BETWEEN x AND y	查找不在该范围内的值
IN()				查找在一组值内的值
NOT IN()			查找不在一组值内的值
IS bool
IS NOT bool			判断布尔值
IS NULL
IS NOT NULL			判断 NULL 值
LIKE 				
NOT LIKE			简单模式匹配（_ 一个任意字符，? 多个任意字符）
```

### 3、逻辑运算符

```
AND &&
OR  ||
NOT !
```

### 4、赋值运算符

```
:=
=		SET 专用，例：SET @name='ycz';
```

### 5、流程控制函数

```mysql
# CASE 语法1（类似switch-case）
CASE value WHEN compare_value THEN result [WHEN compare_value THEN result ...] [ELSE result] END

# CASE 语法2（类似多重if）
CASE WHEN condition THEN result [WHEN condition THEN result ...] [ELSE result] END

# IF/ELSE 如果expr1为真，则返回expr2，否则返回expr3。（类似 ?:）
IF(expr1, expr2, expr3)

# NULL的 IF/ELSE（如果expr1不是 NULL，则返回 expr1，否则返回 expr2）
IFNULL(expr1, expr2)

# 如果 expr1=expr2 则返回 NULL，否则返回 expr1
NULLIF(expr1,expr2)
```

### 6、日期函数

```mysql
# 返回系统时间 格式：2021-09-01 09:00:00
NOW()					# 返回语句开始执行的时间，常量
SYSDATE()				# 返回执行到该处时的实际时间
CURRENT_TIMESTAMP		# NOW() 的别名
CURRENT_TIMESTAMP()		# NOW() 的别名

# 用以下例子查看区别
SELECT NOW(), SLEEP(1), NOW();
SELECT SYSDATE(), SLEEP(1), SYSDATE();

# 返回当前 日期、时间
CURDATE()
CURRENT_DATE		# CURDATE() 的别名
CURRENT_DATE()		# CURDATE() 的别名

CURTIME()
CURRENT_TIME		# CURTIME() 的别名
CURRENT_TIME()		# CURTIME() 的别名

# 获取给定时间 年月日时分秒 
YEAR(date) 
MONTH(date) 
DAY(date) 
HOUR(time) 
MINUTE(time)
SECOND(time)
WEEKDAY(date)	# 星期索引（0：周一，1：周二，...，6：周日）

DATE(expr)		# 获取日期表达式的日期部分
TIME(expr)		# 获取日期表达式的时间部分

# 日期、时间 计算
ADDDATE(date,INTERVAL expr unit)
ADDDATE(date,days)
ADDTIME(expr1,expr2)

SUBDATE(date,INTERVAL expr unit),
SUBDATE(expr,days)
SUBTIME(expr1,expr2)

DATE_ADD(date,INTERVAL expr unit)	# 日期加上指定的时间间隔
DATE_SUB(date,INTERVAL expr unit)	# 日期减去指定的时间间隔

DATEDIFF(expr1,expr2)	# 返回两个日期相差的天数
TIMEDIFF(expr1,expr2)	# 返回两个日期相差的时间数（时:分:秒）

# 日期格式化（%Y-%m-%d %H:%i:%S）
DATE_FORMAT(date,format) 	# 时间表达式格式化
FROM_UNIXTIME(unix_timestamp[,format])	# 时间戳格式化
FROM_DAYS(N)				# 给定一个天数N，返回一个 DATE值
STR_TO_DATE(str,format)		# 将日期格式的字符串转换成指定格式的日期

# 获取当前（或指定）时间的时间戳
UNIX_TIMESTAMP([date]) 
```

### 7、字符串函数

```
①length(str) 获取字符串的字节数

②CONCAT(str1, str2, ...) 拼接字符串
   CONCAT_WS(separator,str1,str2,…) 第一个参数是其它参数的分隔符。分隔符的位置放在要连接的两个字符串之间。如果分隔符为 NULL，则结果为 NULL。

③upper(str) 转换成大写
 lower(str) 转换成小写

④substr(str,start)		截取从起始位置到末尾的字符串
 substr(str,start,len)	截取从起始位置指定长度的字符串
 left(len)		从左边开始截取指定长度
 right(len)	从右边开始截取指定长度
 mid(len)	从中间开始截取指定长度，不指定截取到末尾

⑤instr(str,substr) 返回字串第一次出现的位置，未找到返回0

⑥trim(str)  去字符串两端的空格
   trim(key,str) 去字符串两端的指定字符
   ltrim()
   rtrim()

⑦lpad(str,len,k) 将字符串左填充至指定长度
   rpad(str,len,k) 将字符串右填充至指定长度

⑧replace(str,old,new) 替换字符串

⑩ISNULL(表达式)
# 判断字段或表达式是否为空，如果是返回1，否则返回0
```

### 8、其它函数

```
# 加密函数
MD5()		

# 信息函数
VERSION()	查看数据库版本
DATABASE()	查看当前选中的数据库
USER()		查看当前用户

# 其它
UUID()		生成唯一标识符
SLEEP()		休眠数秒

# 全文检索（配合全文索引使用）
MATCH (col1,col2,...) AGAINST (expr [search_modifier])
```

 

## 五、数据操作语句（DML）

### 1、插入语句

（1）常规插入

```
INSERT INTO 表名(字段名, ...) VALUES(值, ...);

# 批量插入
INSERT INTO 表名(字段名, ...) 
VALUES(值, ...)
VALUES(值, ...)
...
```

（2）蠕虫复制

```
INSERT INTO 表名(字段名, ...)
SELECT 
	字段名, ...
FROM 表名
```

（3）主键冲突

```
# 主键冲突更新
INSERT INTO 表名(字段名, ...) 
VALUES(值, ...) 
ON DUPLICATE KEY UPDATE 字段名=值, ...;

1、存在主键，当主键重复时则执行 UPDATE；
2、不存在主键，存在唯一键，当唯一键重复时则执行 UPDATE；
3、不存在主键，不存在唯一键，存在普通索引，当普通索引重复时则执行 UPDATE；

键冲突比较优先级：
主键 ＞ 唯一键 ＞ 普通索引

# 主键冲突替换
REPLACE INTO 表名(字段名, ...)
VALUES(值, ...);
```

### 2、更新语句

```
UPDATE 表名 SET 字段=值 WHERE 条件 LIMIT 数量;
```

### 3、删除语句

```
DELETE FROM 表名 WHERE 条件 LIMIT 数量;
TRUNCATE 表名;				

# delete 与 truncate 的区别
①delete可以加where条件，truncate不能。
②truncate删除效率稍高一点。
③delete删除后自增长列后，新增记录编号从断点处开始，而truncate从1开始。
④truncate没有返回值，delete有。
⑤truncate删除不能回滚，delete能。
```

### 4、查询语句

> SQL 执行顺序

![SQL执行顺序.png](Images\MySQL_SQL执行顺序.png)

> 查询语法

```mysql
SELECT 
	ALL|DISTINCT
	*
FROM 表名
[[LEFT|RIGHT|INNER] JOIN 表名 
 ON 连接条件]
[WHERE 分组前筛选]
[GROUP BY 分组字段 ASC/DESC] 
[WITH ROLLUP]
[HAVING 分组后筛选]
[ORDER BY 排序字段 ASC/DESC]
[LIMIT 数量]
```

（1）select 选项

```
all			全部（默认）
distinct	去重
as|空格	   取别名
into		变量赋值
```

（2）group by子句（只为统计，查询只取每组第一条数据）

```
# 分组函数（聚合函数）
COUNT() 计算个数
SUM() 求和
MAX() 最大值
MIN() 最小值
AVG() 平均值
GROUP_CONCAT() 合并统计字段

注：①分组函数做条件肯定是放在having子句中
②优先使用分组前筛选
③可以和distinct 搭配使用实现去重，例：count(distinct 字段)

#回溯统计
对已经分组统计的数据再次统计
语法：group_by 分组字段 with rollup

# 多分组：前面的字段先分组，后面的字段再分组

# 分组排序：group by 字段 [desc/asc]
```

（3）limit子句

```
①限制数量 limit 数量
②分页查询
limit [offset,] size;
offset：起始索引（从0开始）
size：显示数量
分页显示：limit (page-1)*size,size;
```

（4）联合查询

```
select 语句；
union [all,distinct]
select 语句；

注：①默认distinct去重，可使用union all 保留重复记录。
②在联合查询中使用order by，必须将select语句括起来。
③要想order by生效，需添加limit语句。
```

（5）连接查询

```
1.SQL92语法
①等值连接
②非等值连接
③自连接

2.SQL99语法
①内连接 [inner] join
②外连接 left/right join
③交叉连接（笛卡尔积） cross join

select *
from 表1 
[join_type] join 表2 
on 连接条件
where 筛选条件

注：①若内连接没有匹配条件就是交叉连接的结果（笛卡尔积）。
②using关键字：表1 [type] join 表2 using(连接字段)；
```

（6）子查询

```
1、按结果集分类
①标量子查询（一行一列）	√
②列子查询		√
   一般搭配关键字使用：in/not in     any     all
③行子查询
  构造行元素：（字段1，字段2）= 行子查询		
④表子查询  

2、按位置分类
①where\having	支持标量子查询和列子查询	√
②from		支持表子查询
③exists（子查询）	判断子查询是否为空，返回1或0。

注：①标量子查询，列子查询、行子查询都属于where子查询。
②表子查询属于from子查询。

3、子查询中的特定关键字
in：where 条件 in （列子查询）
any：=any（列子查询）、<>any（列子查询）
some：与any一样
all：=all（列子查询）、<>all（列子查询）
```

### 5、事务

> 只有 InnoDB 存储引擎支持事务

```sql
# 关闭自动提交
SET AUTOCOMMIT = 0;

# 开启事务 - 可选
START TRANSACTION; 

# 要执行的SQL语句
语句1；
语句2；
......

# 结束事务
# 1-提交事务（同步数据） 
COMMIT;	

# 2-回滚事务（清空之前的操作）
ROLLBACK;	

"""
回滚点 savepoint
记录失败语句之前的位置
增加回滚点：savepoint 回滚点名；
回到回滚点：rollback to 回滚点名；
"""
```

#### 5.1、事务的特性（ACID）

原子性（Atomicity）：事务是一个不可分割的工作单元（整体），确保事务中的所有操作要么全部完成，要么全部不执行。如果事务执行失败，则需要进行回滚。
一致性（Consistency）：事务不能破坏数据的完整性和业务的一致性，确保事务将数据库从一个有效的状态转变到另一个有效的状态。
隔离性（Isolation）：事务所操作的数据在提交之前，对其他事务的可见程度。
持久性（Durability）：一旦事务提交，它对数据库中数据的改变就是永久的。

>   ACID 实现原理：

-   原子性：通过 undolog 实现，需要回滚时，可以利用 undolog 来撤销已执行的操作。
-   一致性：由事务的原子性、隔离性、持久性共同保证。
-   隔离性：隔离级别通过数据库的锁机制（针对写写操作）和 MVCC 技术（针对读写操作）实现。
-   持久性：由 redolog 保证，redolog 被称作重做日志，是物理日志，记录了事务执行后需要重做的操作。redolog 确保所有已提交的事务能够被重做，从而保持数据的持久性。

#### 5.2、隔离级别

```sql
#查看当前的隔离级别：
select @@tx_isolation;

#设置当前连接的隔离级别：
set transaction isolation level read committed;
```

在并发情况下，对同一个数据进行读写操作会产生并发问题。事务的隔离级别用来解决**并发事务**所产生的问题。

-   READ UNCOMMITTED（读未提交）
    
    会出现**脏读**：在事务进行过程中，读到了其它事务未提交的数据。
    
-   READ COMMITED（读已提交）
    
    避免脏读
    
    会出现**不可重复读**：在一个事务过程中，同一行数据多次查询的结果不同（读到其他事务已提交的数据）。
    
-   REPEATABLE READ（可重复读）-- 默认
    
    避免脏读和不可重复读
    
    会出现**幻读**：在一个事务过程中，用同样的操作查询数据（争对行数的变化），得到的记录数不相同。
    
-   SERIALIZABLE（串行化）

    避免幻读，并发性能低（加表锁）。

#### 5.3、MVCC

MVCC（Multiversion Concurrency Control）多版本并发控制：是 InnoDB 引擎用于解决 读写冲突的机制，能在**无锁的情况下支持并发事务**。它通过数据版本管理和 ReadView（读取视图）来实现一致性读，从而避免加锁，提高数据库吞吐量。

1、MVCC 依赖的两个重要机制：

-   版本链（Undo Log）：数据更新不会直接覆盖，而是生成多个历史版本，新旧数据通过 undolog 形成版本链。
-   快照视图（Read View）：事务查询时生成 ReadView，保证事务期间查询的数据始终一致（即一致性读）。

2、MVCC 的关键数据结构：

在 InnoDB 的聚簇索引记录中，每条数据都会有三个隐藏的系统字段，用于 MVCC 版本控制：

| 隐藏列              | 作用                                                         |
| ------------------- | ------------------------------------------------------------ |
| DB_ROW_ID（6 Byte） | 未指定主键时，InnoDB 自动维护的主键。                        |
| DB_TRX_ID（6 Byte） | 标记最后一次修改（删改）该行的事务ID。                       |
| DB_ROLL_PTR         | 回滚指针，指向该行的上一个版本对应的 undolog 地址（回滚日志）。事务回滚时，通过这个指针就能找到历史数据版本。 |

3、版本链（undolog chain）：

每次 UPDATE/DELETE，InnoDB 不会直接修改数据，而是

-   生成 Undo Log，记录旧值。
-   新值更新到数据页。
-   事务提交后，新数据可见，旧数据仍可用于 MVCC 读取。

4、ReadView（快照视图）

ReadView 决定事务在 MVCC 读取哪些版本的数据，它包含：

| 字段             | 含义                                           |
| ---------------- | ---------------------------------------------- |
| m_trx_ids        | 记录当前所有未提交事务的 ID                    |
| m_up_limit_id    | 所有未提交事务 ID 的最小值，即最早的未提交事务 |
| m_low_limit_id   | 下一个将要分配的事务 ID                        |
| m_creator_trx_id | 当前事务的 ID                                  |

Read View 判断 trx_id 对应的记录是否可见的规则：

-   trx_id 小于 m_up_limit_id：当前事务在 ReadView 生成前已提交，**可见**。
-   trx_id 属于 m_trx_ids：当前事务未提交，**不可见**。
-   trx_id 大于等于 m_low_limit_id：当前事务在 ReadView 生成后才开始，**不可见**。
-   trx_id 等于 m_createor_trx_id：事务自己修改的数据，**可见**。

5、MVCC 在不同隔离级别的行为：

-   读未提交：无需锁、无需 MVCC，因为修改操作直接修改数据源，所以会出现脏读。
-   读已提交：每次查询都会创建 ReadView 快照，会导致不可重复读。
-   可重复读：事务第一次查询时创建 ReadView，后续相同查询复用，保证事务期间的一致性读。
-   串行化：MVCC 不生效，使用表锁，并发性低。

6、注意：

-   UndoLog过多影响性能：长事务会导致 Undo Log 不能回收，占用大量存储。
-   事务过多可能影响 MVCC 的查询性能，m_trx_ids 过长，会导致 Read View 计算开销增加。



### 6、锁

（1）表锁

> 偏读型数据库中使用表锁，偏向 MyISAM 存储引擎，开销小，加锁快，无死锁，锁粒度大，发生锁冲突的概率高，并发低。

```mysql
# 查看所有库表是否加锁（In_use = 1 表示已加锁）
SHOW OPEN TABLES;

# 表加锁
LOCK TABLE 表名 READ|WRITE [, 表名 READ|WRITE];

# 表解锁
UNLOCK TABLES;

# 分析锁定表
SHOW STATUS LIKE 'table_locks_%';
/**
 * Table_locks_immediate	产生表级锁定的次数（可以获取到锁定表的数据）
 * Table_locks_waited		出现表级锁定时，等待的次数（不能立即获得锁定表的数据）
 */
```

> 读锁：共享锁，读和读可以共享。

加读锁后，当前连接只能【读】锁定的表，不能读写其它表。其它连接可以读被锁定的表，但写锁定的表时，阻塞。

> 写锁：排它锁，读写互斥、写写互斥。

加写锁后，当前连接能【读、写】锁定的表，不能读写其它表。其它连接读、写锁定表时，阻塞。

注：MyISAM 在执行查询语句前，会给涉及的表加读锁。在执行写操作前，会给涉及的表加写锁。

（2）行锁

> 偏写型数据库中使用行锁，偏向 InnoDB 存储引擎，开销大，加锁慢，会出现死锁，锁定粒度小，发生锁冲突概率低，并发高。
>
> 行锁支持事务。
>
> 行锁可以让读读互斥，可以避免在查询到库存时，库存已经被修改的情况发生。索引失效时，注意锁升级（行锁升级为表锁）。

```mysql
# 加行锁
SELECT ... FOR UPDATE;

# 分析锁定行
SHOW STATUS LIKE 'innodb_row_lock_%';
/**
 * Innodb_row_lock_current_waits	当前正在等待锁定的数量
 * Innodb_row_lock_time				锁定总时长
 * Innodb_row_lock_time_avg			平均锁定时间
 * Innodb_row_lock_time_max			最大锁定时间
 * Innodb_row_lock_waits			锁定总次数
 */
```

（3）间隙锁

在通过范围条件而不是等值条件检索数据时，InnoDB 会锁定范围内的所有索引键值，即使这个键值不存在，被锁定的“间隙”就是间隙锁。此时想要插入间隙内的数据时，无辜阻塞。

共享锁和独占锁

意向锁

记录锁

临键锁

### 7、预处理语句

> MySQL 官方将 PREPAER、EXECUTE、DEALLOCATE 统称为 PREPARE STATEMENT。

```mysql
# 声明预处理语句
PREPAER stmt_name FROM sql_statement;

# 执行预处理语句
EXECUTE stmt_name [USING @a, @b, ...];

# 释放预处理语句
DEALLOCATE PREPARE stmt_name;

# 使用场景
① 当变量做表名时
② 用 ? 作参数占位符
```



## 六、存储引擎

### 1、基本操作

```mysql
# 查看数据库支持的存储引擎
SHOW ENGINES;

# 查看数据库当前使用的存储引擎
SHOW VARIABLES LIKE '%storage_engine%';

# 查看数据库表所用的存储引擎
SHOW CREATE TABLE 表名;

# 创建表时指定存储引擎
CREATE TABLE 表名(col_name col_type) ENGINE = engine_name;

# 修改表的存储引擎
ALTER TABLE 表名 ENGINE = engine_name;
```

> 修改默认的存储引擎

```mysql
# 临时修改
SET default_storage_engine = 'MyISAM/InnoDB';

# 修改配置文件 my.ini 
default-storage-engine = MyISAM/InnoDB;
```

### 2、存储引擎特性对比

|                   | InnoDB   | MyISAM     |
| ----------------- | -------- | ---------- |
| 事务              | 支持     | 不支持     |
| 锁机制            | 行级锁   | 表锁       |
| 索引              | 聚簇索引 | 非聚簇索引 |
| 外键              | 支持     | 不支持     |
| 空间 / 内存使用率 | 高       | 低         |

文件结构

```mysql
# InnoDB存储引擎
    db.opt	字符集
    * .frm	表结构（frame）
    * .ibd	索引+数据（innodb data）
# MyISAM存储引擎
    * .frm	表结构（frame）
    * .MyI	索引（MyISAM Index）
    * .MYD	数据（MyISAM Data）
```

InnoDB 表必须要有主键，且推荐整形自增（减少往索引中间的插入和平衡）。若不存在主键，MySQL 会自动增加一个隐藏列作为主键维护表。

**Table 18.1 Storage Engines Feature Summary**

| Feature                                | MyISAM       | Memory           | InnoDB       | Archive      | NDB          |
| :------------------------------------- | :----------- | :--------------- | :----------- | :----------- | :----------- |
| B-tree indexes                         | Yes          | Yes              | Yes          | No           | No           |
| Backup/point-in-time recovery (note 1) | Yes          | Yes              | Yes          | Yes          | Yes          |
| Cluster database support               | No           | No               | No           | No           | Yes          |
| Clustered indexes                      | No           | No               | Yes          | No           | No           |
| Compressed data                        | Yes (note 2) | No               | Yes          | Yes          | No           |
| Data caches                            | No           | N/A              | Yes          | No           | Yes          |
| Encrypted data                         | Yes (note 3) | Yes (note 3)     | Yes (note 4) | Yes (note 3) | Yes (note 5) |
| Foreign key support                    | No           | No               | Yes          | No           | Yes          |
| Full-text search indexes               | Yes          | No               | Yes (note 6) | No           | No           |
| Geospatial data type support           | Yes          | No               | Yes          | Yes          | Yes          |
| Geospatial indexing support            | Yes          | No               | Yes (note 7) | No           | No           |
| Hash indexes                           | No           | Yes              | No (note 8)  | No           | Yes          |
| Index caches                           | Yes          | N/A              | Yes          | No           | Yes          |
| Locking granularity                    | Table        | Table            | Row          | Row          | Row          |
| MVCC                                   | No           | No               | Yes          | No           | No           |
| Replication support (note 1)           | Yes          | Limited (note 9) | Yes          | Yes          | Yes          |
| Storage limits                         | 256TB        | RAM              | 64TB         | None         | 384EB        |
| T-tree indexes                         | No           | No               | No           | No           | Yes          |
| Transactions                           | No           | No               | Yes          | No           | Yes          |
| Update statistics for data dictionary  | Yes          | Yes              | Yes          | Yes          | Yes          |

## 七、分区

```sql
# 查看是否支持分区
SHOW PLUGINS;
partition	ACTIVE	...
```

分区方式：

- Range：基于连续区间分区
- List：基于离散集合分区
- Hash：根据用户自定义规则对数据进行散列分区
- Key：与 hash 分区类似，但是根据 MySQL 引擎自身的规则处理数据

#### （1）Range

```sql
# Range
CREATE TABLE `part_range` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `data` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE (id) (
  PARTITION p0 VALUES LESS THAN (10) ENGINE = InnoDB,
  PARTITION p1 VALUES LESS THAN (20) ENGINE = InnoDB,
  PARTITION p2 VALUES LESS THAN (30) ENGINE = InnoDB
);

# 多次执行
INSERT INTO `part_range` VALUES();

# 执行到第 30 次时，报错
> 1526 - Table has no partition for value 30

# 增加分区
ALTER TABLE part_range ADD PARTITION(PARTITION p3 VALUES LESS THAN MAXVALUE);

# 再次插入即可成功

# 按分区查询
SELECT * FROM part_range PARTITION(p3);
```

#### （2）List

```sql
# List
CREATE TABLE `part_list` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `data` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY LIST (id%10) (
  PARTITION p0 VALUES IN (1,3,5,7,9),
  PARTITION p1 VALUES IN (2,4,6,8,0)
);
```

#### （3）Hash

Hash 分区的特点：

- 根据 MOD(分区键, 分区数) 的值把数据行存储到表的不同分区中。
- 数据可以平均的分布在各个分区中。
- Hash 分区的键值必须是一个 int 类型的值，或通过函数可以转为 int 类型。

```sql
# Hash：id % 4
CREATE TABLE `part_hash` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `data` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY HASH (id) PARTITIONS 4;
```

#### （4）Key

Key 分区类似于 Hash 分区，不同之处在于 Hash 分区使用用户自定义的表达式，而 Key 分区的散列函数由 MySQL 服务器提供。NDB 集群使用 MD5() 来实现此目的。对于其它存储引擎的表，服务器采用其自身的内部散列函数。

Key 分区只接受零或多个列名的列表。任何用作分区键的列都必须包含表的部分或全部主键（若有）。如果没有将列名指定为分区键，则使用表的主键（若有）。

```sql
# Key
CREATE TABLE `part_key` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `data` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY KEY (id) PARTITIONS 4;
```

### 分表 vs 分区

分表后，单表的并发能力提高了，磁盘 IO 性能也提高了。分表的侧重点是存取数据时，如何提高 MySQL 的并发能力。

分区后，主要突破磁盘 IO 瓶颈，提升磁盘的读写能力，来增加 MySQL 的性能。分区的侧重点是提高磁盘的读写能力。





## 八、存储过程

```
========== 变量 ==========
一、系统变量（全局变量，会话变量）
①查看所有的系统变量：show global | session variables；
②查看满足条件的部分变量：show global | session variables like '%char%';
③查看指定的某个系统变量的值 select @@global | session. 系统变量名
④为某个系统变量赋值：set global | session 系统变量名 = 值 ；
	set @@global  | session . 系统变量名 = 值 ；
注：如果是全局级别，需要加global，如果是会话级别，则需加session，不加关键字默认为会话变量。
局部修改（会话级别）：只针对当前客户端的当次连接有效。
全局修改：针对所有的客户端，所有时刻有效。全局修改之后，当前连接无效，只对新开连接有效。

二、自定义变量
1、用户变量
作用域：针对当前会话（连接）有效，与会话变量的作用域相同
①声明并初始化
set @用户变量名=值
set @用户变量名:=值
select @用户变量名:=值
②赋值（变量专用赋值号 := ）
方式一：
set @用户变量名=值
set @用户变量名:=值
方式二：
select @用户变量名:=值 from 表；
select 字段 into @用户变量名 from 表；
③查看
select @用户变量名；

2、局部变量（begin-end 内有效，declare关键字声明）
①声明并初始化
declare 局部变量名 类型；
declare 局部变量名 类型 default 值；
②赋值
方式一：
set 局部变量名=值
set 局部变量名:=值
select 局部变量名:=值
方式二：
select 字段 into 局部变量名 from 表；
③查看
select 局部变量名；

三、变量作用域
1、局部变量
  使用declare关键字声明，只能存在于结构体内部（函数、存储过程、触发器）
2、会话变量
  在当前用户的当前连接有效，只要在本连接中，任何地方都可以使用（可以在结构体内部，也可以跨库）
3、全局变量
  所有客户端所有连接都有效，需要使用全局符号定义。
  set global 系统变量名 = 值 ；
  set @@global . 系统变量名 = 值 ；
通常在SQL编程中，不会使用自定义变量来控制全局，一般都是定义会话变量或局部变量来解决问题。

========== IF 分支结构 ========== 
1、简单if结构（类似PHP三目运算符）
IF(条件表达式，为真时结果，为假时结果);

2、单分支结构：
IF(条件)
THEN
     为真时，执行语句;
ELSE
     为假时，执行语句;
END IF;

3、多分支结构：
IF(条件1)
THEN
     语句1;
ELSE IF(条件2)
THEN
     语句2;
.......

ELSE
     语句n;
END IF;

========== 循环结构 ========== 
1、WHILE 循环
while(条件) do
begin
	条件为真时，执行语句。。。
end;
end while;

2、REPEAT循环
repeat
begin
	语句。。。
end;
until 条件
end repeat;

3、while 结构标识符
作用：循环控制
标识名:while 条件 do
  if 条件 then
    iterate/leave 标识名；
  endif；
  循环体；
endwhile[标识名]；
注：
iterate <=> continue
leave <=> break

========== 游标 ========== 
功能：游标用于遍历查询结果集
语法：
BEGIN
	# 游标循环变量
	DECLARE i INT DEFAULT 0;
	# 定义游标，并将sql结果集赋值到游标中
	DECLARE ret_cur CURSOR FOR 查询语句;
	# 声明当游标遍历完后将标志变量置成某个值
	DECLARE CONTINUE HANDLER FOR found SET i=1;

	# 打开游标
	OPEN ret_cur;
	# 取出第一条游标值，赋值给变量
	FETCH ret_cur INTO 变量1, 变量2, ... ;
	
	WHILE(i=0) DO
		执行业务逻辑语句;
		# 取出下一条游标值，
		FETCH ret_cur INTO 变量1, 变量2, ... ;
	END WHILE;
	
	# 关闭游标
	CLOSE ret_cur;
END;


========== 存储过程 ========== 
1、创建语法
CREATE PROCEDURE 存储过程名([参数模式 参数名 类型, ...])
BEGIN
	存储过程体
END

参数模式：
IN：该参数可以作为输入，也就是该参数需要调用处传入值
OUT：该参数可以作为输出，也就是该参数可以作为返回值
INOUT：该参数既可以作为输入又可以作为输出

注：
①OUT类型的参数传入后首先会被清空（形参被改变，实参实际没变）。
②过程结束，会将形参的最终值重新赋值给实参（实参被改变）。


存储过程体中的每条SQL语句的结尾要求必须加分号。
存储过程的结尾可以使用 delimiter 重新设置
语法：delimiter 结束标记

2、调用语法
call 存储过程名（实参列表）


========== 自定义函数 ========== 
# 查询函数开启状态
show variables like 'log_bin_trust_function_creators';
# 开启二进制日志：
set global log_bin_trust_function_creators=1;
# 修改语句结束符
delimiter $$

1、创建函数语法：
CREATE FUNCTION 函数名([参数1 类型, ...])
RETURNS 返回值类型	-- 注意是：RETURNS
BEGIN
	#函数体#
	return 0;
END
2、调用语法：
select 函数名(参数);

# 查看所有自定义函数
show function status;

注：
①自定义函数属于用户（会话）级别的，只有当前客户端对应的数据库中可以使用。
②可以在不同的数据库下看到对应的函数，但不可以调用。
③为规范函数返回值，在函数内部不能使用select指令查看数据，因为select返回的是一个结果集（set）。

3、应用
① 生成随机字符串的函数
CREATE FUNCTION rand_str(n INT) 
RETURNS VARCHAR(255)
BEGIN
	DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
	DECLARE return_str VARCHAR(255) DEFAULT '';
	DECLARE i INT DEFAULT 0;
	WHILE i < n DO
	SET return_str = CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));
	SET i = i + 1;
	END WHILE;
	RETURN return_str;
END

② 生成5位随机数的函数
CREATE FUNCTION rand_num()
RETURNS INT(5)
BEGIN 
	DECLARE i INT DEFAULT 0;
	SET i = FLOOR(100+RAND()*10);
	RETURN i;
END 


========== 函数与存储过程的区别 ==========
①存储过程与函数都是一次编译，后续执行。
②函数中必须有返回值，而存储过程没有返回值。
③存储过程没有返回值类型，不能将结果赋值给变量；函数有返回值类型，调用时，除在select中，必须将返回值赋值给变量。
④函数可以在select语句中直接使用，而存储过程不能。
⑤调用方式不同。


========== MySQL的触发器 ==========
概念：触发器是一种特殊类型的存储过程，触发器是通过事件进行触发而被执行。

用途：①可在写入数据表前，强制检验或数据类型转换。
②触发器发生错误时，异动的结果会被撤销。（如果触发器执行错误，那么前面用户已经执行成功的操作也会被撤销）
③在修改一张数据表的同时更新其他相互关联的数据表。
④有时候关联的数据表非常多，能让MySQL自动帮我们完成。

记录关键字：new、old
old：保存操作前的数据；
new：保存操作后的数据。

insert 操作前没有数据，因此只有new；
update 操作有old和new；
delete 操作只有new。

#插入一条数据的触发器
begin
update tb_user set count_fan=count_fan+1 where id = new.up_uid;
update tb_user set count_follow=count_follow+1 where id=new.fan_uid;
insert into tb_dt (uid,content) values(new.fan_uid,'我新关注了一个up主');
end

#删除一条数据的触发器
begin
update tb_user set count_fan=count_fan-1 where id=old.up_uid;
update tb_user set count_follow=count_follow-1 where id=old.fan_uid;
end

# 防止库存超卖
①订单数据插入“前”创建触发器；
②判断库存数量；
③超卖时，主动出错。

```



## 九、其它

```
========== 表关系及业务处理 ==========
1、一对一
  一张表中的一条记录与另外一张表中最多一条记录相关联，通常，让两张表中使用同样的主键即可。
2、一对多
  在“多”关系的表中维护“一”关系表的主键。
3、多对多
  使用第三张表来存储两张表的关系。



========== MySQL数据库备份与还原 ==========
mysqldump.exe  SQL备份客户端

# root用户重置密码
1、关闭服务 net stop mysql
2、重启服务：
mysqld.exe --skip-grant-TABLEs	// 启动服务器，跳过权限
3、当启动的服务器没有权限概念时，非常危险，任何客户端不需要任何用户信息都可以直接登录，而且是root权限。
新开客户端，使用mysql.exe登录即可。
4、修改root用户的密码
update mysql.user set password = password('root') where user='root' and host = 'localhost';
5、结束服务，重启服务。

# 查看服务器插件信息
SELECT * FROM INFORMATION_SCHEMA.PLUGINS;
SHOW PLUGINS;
注：	
	PLUGIN_LIBRARY值为 的 NULL都是内置的，无法卸载。
	Library值为 的NULL 都是内置的，无法卸载。
	
	
	
	
小表驱动大表：
in 与 exists 的应用
```



## 十、索引

### 1、CREATE INDEX 语句创建索引

```
CREATE [unique|fulltext|spatial] INDEX 索引名
[using btree|hash]
ON 表名( 索引列[(length)] [asc|desc], ... )

[unique|fulltext|spatial]： 索引类型
[using btree|hash]： 索引数据类型
length：表示使用索引列的前多少个字符创建索引
[asc|desc]：表示索引字段值的排序顺序，默认为asc
索引名称最长64位字符

例1：给Student表的Sname字段创建索引
CREATE INDEX idx_Sname on Student(Sname);

例2：在Score表的StuID和CourseID两个字段创建组合索引
CREATE INDEX idx_stuid_courid ON Score(StuID,CourseID);

(StuID,CourseID)
按StuID单列查可以
按S图ID，CourseID双列查也可以
不能按CourseID单列查的问题
```

### 2、ALTER TABLE 语句创建索引

```
ALTER TABLE 表名 ADD [索引类型] INDEX [索引名] ON (索引字段(length))
给字段加主键、唯一性约束、外键都会创建索引，如下：
主键：ADD [constraint 约束名] PRIMARY KEY [索引类型](主键字段)
唯一性：ADD [constraint 约束名] UNIQUE [索引名][索引类型](主键字段)
外键：ADD [constraint 约束名] FOREIGN KEY [索引类型](主键字段)

例3：为教师表teacher的生日字段增加普通索引，并以降序排序。
ALTER TABLE teacher ADD INDEX USING btree(t_birthday desc);

例4：为专业表major的专业名称字段增加唯一性索引。
ALTER TABLE major ADD UNIQUE idx_major_name (major_name);
```

### 3、CREATE TABLE创建索引

```

```

### 4、删除索引

```
DROP INDEX [索引名称] ON 数据表;
```

### 5、查看索引的命令

```
SHOW INDEX FROM 表名;
```

### 6、索引的数据结构：

- 二叉树

缺点：索引顺序递增时，退化成单边链表。

- 红黑树

二叉平衡树（自动平衡），缺点：层级太多。

横向扩容 => B-Tree

- Hash表

对索引字段进行 hash 运算，hash 结果就是数据行的地址。

缺点：范围查找失效。

- B+Tree

根节点常驻内存；

非叶子节点不存储 data ，只存储索引（冗余），可以放更多的索引；

叶子节点包含所有索引字段，并存储 data （数据内存地址或整行数据）；

叶子节点其实是双向链表，提高区间访问性能；

非叶子节点的大小：默认16KB（可选：4、8、16、32、64 KB）， `SHOW GLOBAL STATUS LIKE 'Innodb_page_size'` ；

非叶子节点能存储的元素个数：16KB /（bigint 8B + 指针 6B）= 1170 个；

![B+Tree_structure](Images\MySQL_B+Tree_数据结构.png)

- B-Tree

![B-Tree_structure](Images\MySQL_B-Tree_数据结构.png)

#### 最左前缀原则

联合索引排序规则：优先按第一个字段排序，相等时，按第二个字段排序。以此类推。

![1632902404562](Images\MySQL_Join-Index_数据结构.png)

最左前缀：若未使用第一个字段，第二个字段无法顺序获取，走全表扫描；

范围之后全失效：第一个值取为区间，第二个值无法顺序排序，此时还是得全表扫描；



## 十一、MySQL 客户端程序

### 1、mysql（MySQL 命令行客户端）

```
mysql [OPTIONS] [database]

选项：
	-u	用户名
	-p	密码
	-h	域名或IP
	-P	端口
	-e	执行SQL语句并退出
	
例：
	mysql -uroot -proot db_name -e "sql statement";
```

### 2、mysqladmin（MySQL 服务器管理程序）

```
mysqladmin [OPTIONS] command command ...

选项：
	-u	用户名
	-p	密码
	-h	域名或IP
	-P	端口
	
例：
	mysqladmin -uroot -proot db_name -e "sql statement";
```

3、mysqlcheck（表的维护程序）

### 4、mysqldump（数据库导出程序）

```
mysqldump [OPTIONS] database [tables]
mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]
mysqldump [OPTIONS] --all-databases [OPTIONS]

选项：
	-u	用户名
	-p	密码
	-h	域名或IP
	-P	端口
	
	-n	不包含数据库的创建语句
	-t	不包含数据表的创建语句
	-d	不包含数据
	
	-T	自动生成两个文件：.sql 表结构，.txt 数据文件
	
	--add-drop-table		在建表前加上 drop table 语句（默认）
	--skip-add-drop-table	不加 drop table
	
例：
	mysqldump -uroot -proot db_name > test.sql;
	mysqldump -uroot -proot -T /tmp db_name
```

5、mysqlpump（数据库备份程序）

### 6、mysqlimport（数据库导入程序）

> 导入 mysqldump -T 导出的[表结构]

```
mysql> source /tmp/test.sql
```

> 导入 mysqldump -T 导出的[数据文件]

```
mysqlimport [OPTIONS] database textfile ...

选项：
	-u	用户名
	-p	密码
	-h	域名或IP
	-P	端口
	
例：
	mysqlimport -uroot -proot test test.txt	
```

### 7、mysqlshow（显示数据库、表和列信息）

```
mysqlshow [OPTIONS] [database [table [column]]]

选项：
	--count	显示数据库及表的统计信息
	-i		显示指定数据库或数据库的状态信息
	
例：
	mysqlshow -uroot -proot --count
```

8、mysqlslap（敷在模拟客户端）

### mysqlbinglog

```
mysqlbinlog [options] log-files

选项：
	-d	数据库
	-o	忽略日志中的前n行命令
	-r	将日志输出到指定文件
	-s	显示简单格式
	--start-datatime
	--stop-datatime
```



## 十二、MySQL 服务器

### 1、日志

> 默认情况下，MySQL 不启用任何日志。调试模式下，需手动开启。

（1）错误日志

> 默认开启

```mysql
# 查看日志文件位置
SHOW VARIABLES LIKE 'log_error';
```

（2）全局查询日志

>  默认情况下，全局查询日志处于禁用状态。

```mysql
# 查看全局查询日志是否开启
SHOW VARIABLES LIKE 'general_log';
SET GLOBAL general_log = ON;

# 查看全局查询日志文件的位置
SHOW VARIABLES LIKE 'general_log_file';
SET GLOBAL general_log_file = '/tmp/general.log';

# 设置日志存储类型（TABLE：`mysql`.`general_log`）
SET GLOBAL log_output = 'FILE|TABLE';
```

（3）二进制日志

> 二进制日志记录了所有的 DDL 和 DML 语句，但不包含查询语句。
>
> 主从复制就是通过 binlog 实现的。

```mysql
# 开启 binlog 日志，日志前缀为 mysql-bin，如：mysql-bin.000001
log_bin=mysql-bin

# 二进制文件格式
binlog_format=STATEMENT|ROW|MIXED

/**
 * STATEMENT：该日志格式在日志文件中记录的都是SQL语句，通过 mysqlbinlog 工具查看。
 * ROW：该日志格式在日志文件中记录的是每一行的数据变更。
 * MIXED：默认格式，混合以上两种格式。
 */
```

> 日志读取
>
> MySQL的数据存储目录：`/var/lib/mysql`

```
mysqlbinlog /var/lib/mysql/mysql-bin.0000001

# 若为 ROW 格式，则需加 -vv 选项
mysqlbinlog -vv /var/lib/mysql/mysql-bin.0000002
```

> 日志删除

```mysql
# 删除全部日志
RESET MASTER;

# 删除指定编号之前的所有日志
PURGE MASTER LOGS TO 'mysql-bin.******';

# 删除指定日期之前的所有日志
PURGE MASTER LOGS BEFORE 'YYYY-mm-dd HH:ii:ss';

# my.conf 配置日志的有效期，
--expire_logs_days=3
```

（4）慢查询日志

```mysql
# 查看慢查询日志是否开启
SHOW VARIABLES LIKE 'slow_query_log';

# 开启慢查询
SET GLOBAL slow_query_log = ON;

# 查看慢查询日志文件的位置
SHOW VARIABLES LIKE 'slow_query_log_file';

# 设置慢查询日志文件位置
SET GLOBAL slow_query_log_file = '/tmp/slow.log';

# 设置慢查询时长
SHOW VARIABLES LIKE 'long_query_time';
SET @@long_query_time = 1

# 记录没有索引的查询
SHOW VARIABLES LIKE 'log_queries_not_using_indexes';
SET GLOBAL log_queries_not_using_indexes = ON;

# 查看日志中有多少条慢查询
SHOW GLOBAL STATUS LIKE 'Slow_queries';

# 设置日志存储类型（TABLE：`mysql`.`slow_log`）
SET GLOBAL log_output = 'FILE|TABLE';
```

> 写入配置文件：永久生效

```ini
slow_query_log=ON;
slow_query_log_file=/tmp/slow.log;
long_query_time=3;
log_output=FILE;
```

> 日志分析工具 mysqldumpslow

```
mysqldumpslow [ OPTS... ] [ LOGS... ]

选项：
	-s	以何种方式排序（默认：at）
        al: average lock time
        ar: average rows sent
        at: average query time
        c: count
        l: lock time
        r: rows sent
        t: query time
    -t	返回前多少条数据
    -g	正则匹配查找，不区分大小写

例：
# 查看查询时间最长的前十条日志
mysqldumpslow -s t -t 10 /var/lib/mysql/localhost-slow.log
```

（5）DDL 日志

（6）InnoDB 日志

undolog

redolog

### 2、SHOW PROFILE

> SHOW PROFILE 表示当前会话过程中执行的语句资源使用信息。
>
> SHOW PROFIEL 即将弃用，[使用性能模式的查询分析](https://dev.mysql.com/doc/refman/5.7/en/performance-schema-query-profiling.html) 。

（1）开启 profile

```mysql
# 查看 profile 状态（默认 OFF）
SHOW VARIABLES LIKE 'profiling';
SET profiling = ON;
```

（2）显示发送到服务器的最新语句的列表

```mysql
# 服务器默认会记录最新的 15 条SQL语句
SHOW VARIABLES LIKE 'profiling_history_size';

# 语法
SHOW PROFILES;
```

| Query_ID | Duration | Query    |
| -------- | -------- | -------- |
| 查询ID   | 持续时间 | 查询语句 |

（3）显示有关单个语句的详细信息

> 默认情况下，SHOW PROFILE 只显示 Status 和 Duration 列。

```mysql
# 语法
SHOW PROFILE [type [, type] ... ]
    [FOR QUERY ID]
    [LIMIT row_count [OFFSET offset]]

/*
TYPE:
    ALL 		显示所有信息
    BLOCK IO	显示块输入和输出操作的计数
    CONTEXT SWITCHES 显示自愿和非自愿上下文切换的计数
    CPU			显示用户和系统 CPU 使用时间
    IPC			显示发送和接收消息的计数
    MEMORY		显示内存使用信息
    PAGE FAULTS	显示主要和次要页面错误的计数
    SOURCE		显示源代码中函数的名称，以及函数所在文件的名称和行号
    SWAPS		显示交换计数
*/   
```

Status 结果分析：出现以下情况说明效率差

converting HEAD to MyISAM：查询结果太大，内存不够用，往磁盘上写。

Creating tmp table：创建临时表。

Copying to tmp table on disk：把内存中的临时表复制到磁盘。

locked：锁。

### 3、主从复制

**复制过程：**

- master 将改变记录到二进制日志（binary_log）。这些记录过程叫二进制日志事件（binary_log_events）；
- slave 将 master 的 binary_log 拷贝到它的中继日志（relay_log）；
- slave 重做中继日志中的事件，将改变应用到自己的数据库中。

**说明：**

MySQL 主从复制是异步且串行化的。

每个 master 可以有多个 salve；

每个 slave 只能有一个 master；

每个 slave 的服务器 ID 必须唯一；

**配置：**

① 主从服务器的配置

```
vi /etc/my.cnf

# 主从数据库的唯一标识
server-id = 1
# 主从服务的核心 log-bin 日志
log-bin = mysql-bin
```

重启服务器

```
service mysqld restart
```

② 主从服务器中的表结构要保持一致。

③ 主服务器配置

创建一个专门用来同步数据的账号

```
grant replication slave on *.* to 'sync'@'%' identified by 'Pwd-123456';
```

查看状态

```
show master status;
```

| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
| :--------------- | -------- | ------------ | ---------------- | ----------------- |
| mysql-bin.000001 | 433      |              |                  |                   |

④ 从服务器配置	

> 要与 show master status; 的结果一致		

```mysql
change master to master_host='192.168.56.103', master_user='sync', master_password='Pwd-123456', master_log_file='mysql-bin.000001', master_log_pos=433;
```

开启从服务

```
start slave;
```

查看从服务状态

```
show slave status;

# 以下值为 YES 则配置成功
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

⑤ 测试：在主服务器上插入数据，从服务器上自动同步，OK。



## 十三、SQL 优化

### 1、SQL 执行频率

> 默认查看当前连接（SESSION）的执行频率

```mysql
SHOW [GLOBAL|SESSION] STATUS LIKE 'Com_%';
```

> 针对 innodb

```mysql
SHOW [GLOBAL|SESSION] STATUS LIKE 'Innodb_rows_%';
```

### 2、定位低频 SQL

```mysql
SHOW PROCESSLIST;
```

### 3、EXPLAIN 执行计划

EXPLAIN 可以模拟执行 SQL 查询语句，从而知道 MySQL 是如何处理你的 SQL 语句的。分析你的查询语句或表结构的性能瓶颈。

> EXPLAIN 性能指标参数：

```sql
EXPLAIN SELECT * FROM table WHERE 1 = 1;
```

| 字段          | 说明                                                         |
| :------------ | :----------------------------------------------------------- |
| id            | 执行顺序标识符，id 越大优先级越高，id 相同，从上往下执行。   |
| select_type   | 查询类型：<br/>**SIMPLE**：简单查询，不包括连接查询和子查询。<br/>**PRIMARY**：主查询，通常与 SUBQUERY、DERIVED、UNION 一起出现。<br/>**SUBQUERY**：子查询。<br/>**DERIVED**：衍生查询，得到最终查询结果前会用到临时表。<br/>**UNION**：联合查询，第一个表为 PRIMARY，后面的表为 UNION。<br/>**UNION RESULT**：使用 UNION 会出现 UNION RESULT 对结果去重，使用 UNION ALL 时不会出现。 |
| table         | 查询的表                                                     |
| partition     |                                                              |
| type          | 访问类型（MySQL 在表中找到所需行的方式）：<br/>system > const >eq_ref > ref > range > index > ALL ＞ NULL <br/><br/>**NULL**：MySQL 在优化过程中分解语句，执行时甚至都不用访问表或索引，例如从一个索引列里选取最小值（通过单独索引查找即可完成）。<br/>**system**，**const**：当 MySQL 对查询部分进行优化，并转换为一个常量时，使用这类型访问。如 [Primary Key] 查找，system 是 const 的特例，当查询的表只有一行的情况下使用。<br/>**eq_ref**：类似 ref，区别在于使用的索引时唯一索引。常见于多表连接中使用 [Primary Key] 或 [Unique Key] 作为关联条件。<br/>**ref**：查询用到了非唯一索引（常与 Extra：Using index condition 一起出现）。<br/>**range**：范围查询，只使用一个索引来检索给定范围的行。<br/>**index**：Full Index Scan，index 与 all 的区别：index 类型只遍历索引树（常与 Extra：Using index 一起出现）。<br/>**all**：Full Table Scan 全表扫描。 |
| possible_keys | 可能使用的索引。                                             |
| key           | 实际使用的索引，如果为 NULL，则没有使用索引。<br/>在查询中使用 `FORCE INDEX`、`IGNORE INDEX`、`USE INDEX` 可以强制 MySQL 使用索引和忽视 possible_type 中的索引。 |
| key_len       | 表示 MySQL 选择的索引字段的长度（Byte）。通过 key_len 可以确定 mysql 将实际使用复合索引中的几个字段。 |
| ref           | 哪些列或常量被用于查找索引列上的值。                         |
| rows          | 查询时必须检查的行数。                                       |
| filtered      | 通过查询条件获取的最终记录数 ÷ 通过访问类型（type）指定的方式搜索出来的记录数×100% <br/>如果比例很低，说明存储引擎层返回的数据需要经过大量过滤，效率低。 |
| Extra         | Extra：处理查询时的额外信息。<br/><br/>**Using filesort**【慢】：MySQL中无法利用索引完成的排序，只能在内存或磁盘中进行排序操作。<br/>**Using temporary**【慢】：使用临时表，MySQL 如果不能有效利用索引来完成去重、排序、分组等查询，将创建内部临时表来保存中间结果。<br/>**Using index**：使用索引覆盖，扫描索引树一步完成，不需要回表。<br/>**Using index condition**：使用了索引下推，还需要回表。<br/>**Using where**：使用 WHERE 条件过滤。<br/>**Using join buffer**：使用了连接缓存，在连接查询执行过程中，当被驱动表不能有效的利用索引加快访问速度，MySQL 一般会为其分配一块名叫 join buffer 的内存块来加快查询速度。<br/>**impossible where**：WHERE 子句的值总是 false，不能获取到数据。<br/>**select tables optimized away**：在没有 GROUP 子句的情况下，基于索引优化 MIN、MAX 操作或者对于 MyISAM 存储引擎优化 COUNT(*) 操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。 <br/>**distinct**：优化 DISTINCT，在找到第一个匹配的元组后记停止找相同值的工作。<br />**no matching row in const table**：JOIN 表中没有找到匹配的行。<br />**unique row not found**：JOIN 表中主键或唯一键不存在。 |

> `Using temporary` 产生的条件：

- GROUP BY 的列没有用到索引。
- DISTINCT 的列没有用到索引。
- GROUP BY 的列有索引，ORDER BY 的列没有索引。
- GROUP BY 的列和 ORDER BY 的列不一样，即使都有索引也会产生临时表。
- GROUP BY 或 ORDER BY 的列不是来自 JOIN 语句的第一个表（驱动表）。

注：连接查询的 Explain 结果中第一行出现的表就是驱动表。



索引最左原则：

```
# SQL 语句执行时的最左原则
WHERE ＞ GROUP BY ＞ ORDER BY

# WHERE 条件最左原则
索引顺序：主键 ＞ 唯一索引 ＞ 普通索引 ＞ 无索引
范围顺序：等号 ＞ IN ＞ 范围（>、<、BETWEEN、LIKE）

# 联合索引的最左原则
# 将区分度最高的放在最左边
创建联合索引 (a, b, c)
将生成以下索引
(a)
(a, b)
(a, b, c)
```



```
防止索引失效口诀：
全职匹配我最爱，最左前缀要遵守。
带头大哥不能死，中间兄弟不能断。
索引列上少计算，范围之后全失效。
LIKE百分写最右，覆盖索引不写 * 。
不等空值还有OR，索引影响要注意。
VAR引号不可丢，SQL优化有诀窍。

排序优化：
排序字段也要按索引规则。
sort_buffer_size
max_length_for_sort_data

分组字段也要按索引规则，可以使用 order by null 禁用排序。

连接时，副表的条件字段需要加上索引。
索引列上不能计算、不能进行类型转换。
用 union 代替 or：You can also solve the problem efficiently by using a UNION that combines the output of two separate SELECT statements. See Section 13.2.9.3, “UNION Clause”.

Each SELECT searches only one key and can be optimized:
用 join 代替 子查询
使用 UNION ALL 代替 UNION
```

### 索引下推

索引下推（Index Condition Pushdown, ICP）是 MySQL 优化器在 InnoDB 存储引擎中引入的一种查询优化技术，从 MySQL 5.6 开始支持。它的目的是通过将部分查询条件“下推”到存储引擎层，在扫描索引时就进行过滤，减少数据的回表次数，提升查询性能。

在多列联合索引中尤为有效，是优化复杂查询的一个重要工具。

**索引下推的原理：**

-   传统方法：MySQL 在存储引擎中扫描索引后，返回主键值给服务器层，服务器层再回表获取完整行数据并进行条件过滤。

-   索引下推：MySQL 将部分查询条件下推到存储引擎，存储引擎在扫描索引时即可过滤掉不符合条件的记录，减少回表次数。

**索引下推的适用场景：**

-   多列联合索引：部分查询条件可在索引扫描时应用。
-   查询过滤较强：索引中字段的过滤条件能显著减少数据量。
-   大数据量查询：索引下推的优化效果在大数据量查询中更为明显。



-   索引下推需要查询条件中的字段在索引中存在，并符合最左前缀原则。
-   如果下推的条件无法有效过滤数据，则优化效果不明显。

**使用场景限制：**

-   适用于range、ref、eq_ref和ref_or_null查询
-   InnoDB和MyISAM都支持，Mysql partition分表也可以使用
-   对于InndoDB而言，ICP只支持二级索引，主键不需要回表
-   子查询不支持



### 4、SHOW PROFILE 分析 SQL

```mysql
SHOW PROFILE;
```

### 5、trace 优化器执行计划

> MySQL 5.6 以上

```mysql
# 开启 trace
set optimizer_trace = "enabled=on";

# 设置格式为 JSON
set end_markers_in_json=on;

# 最大内存限制
set optimizer_trace_max_mem_size=1000000;

# execute sql

# 查看执行计划详情
select * from information_schema.optimizer_trace;
```

### 查询缓存优化

```mysql
# 查看是否支持查询缓存
SHOW VARIABLES LIKE 'have_query_cache';

# 查看是否开启了查询缓存
SHOW VARIABLES LIKE 'query_cache_type';

# 查看查询缓存的大小限制
SHOW VARIABLES LIKE 'query_cache_size';

# 查看查询缓存的状态
SHOW STATUS LIKE 'Qcache_%';
```

|                         | 说明                                 |
| ----------------------- | ------------------------------------ |
| Qcache_free_blocks      | 查询缓存的可用内存块数               |
| Qcache_free_memory      | 查询缓存的可用内存量                 |
| Qcache_hits             | 查询缓存命中数                       |
| Qcache_inserts          | 添加到查询缓存的查询数               |
| Qcache_lowmem_prunes    | 因内存不足而从查询缓存中删除的查询数 |
| Qcache_not_cached       | 非缓存查询的数量                     |
| Qcache_queries_in_cache | 查询缓存中注册的查询数               |
| Qcache_total_blocks     | 查询缓存中的块总数                   |

### 内存优化

```

```

### 并发参数

```
# 最大连接数，默认：151
max_connections

# 积压请求栈大小，默认：80
back_log

# 线程可打开表缓存的数量
table_open_cache

# 客户服务线程的缓存数
thread_cache_size

# 行锁等待时间，默认：50ms
innodb_lock_wait_timeout
```



```
========== MySQL调优 ==========
一、性能监控
1、使用 show profile 查询剖析工具，可以指定具体的type。
2、使用performance schema 来更加容易的监控mysql。
3、使用show processlist 查看连接的线程个数，监测线程状态。

二、schema与数据类型优化
1、数据类型优化。（小、简单、非空）
2、合理使用范式和反范式。
3、主键的选择。
4、字符集的选择。
5、存储引擎的选择。
6、适当的数据冗余。
7、适当拆分。

三、执行计划（explain）

四、通过索引进行优化

五、查询优化

六、分区表

七、服务器参数配置



```

### MySQL 错误码

```
# 外键名冲突
MySQL 1022 - Can't write; duplicate key in table '#sql-998_1d24672'

# 等待其它事务释放锁超时
Error 1205: Lock wait timeout exceeded; try restarting transaction

# 语句太长
Error 1390: Prepared statement contains too many placeholders

# 列数太多，列的数据类型占用的空间太大时
> 1118 - Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs

# 字段超过索引长度
> 1170 - BLOB/TEXT column '' used in key specification without a key length
```

### 深分页查询优化

```sql
# 利用延迟关联或者子查询优化深分页
SELECT t1.* FROM 表1 as t1 , (select id from 表1 where 条件 LIMIT 20 OFFSET 1000000) as t2 where t1.id = t2.id
```

### 分表

```sql
CREATE TABLE `member_1` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL DEFAULT '',
  `sex` tinyint(3) unsigned NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM CHARSET=utf8mb4;

CREATE TABLE `member_2` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL DEFAULT '',
  `sex` tinyint(3) unsigned NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM CHARSET=utf8mb4;

CREATE TABLE `member` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL DEFAULT '',
  `sex` tinyint(3) unsigned NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=MERGE UNION=(member_1, member_2) INSERT_METHOD=NO CHARSET=utf8mb4;
# INSERT_METHOD 表示该表的插入方式
#  - NO     只能查询，不能写入
#  - FIRST  插入到第一张表（member_1）
#  - LAST   插入到最后一张表（member_n）

INSERT INTO member_1(`name`, sex) VALUES('Yan1', 1);
INSERT INTO member_2(`name`, sex) VALUES('Yan2', 2);

SELECT * FROM member_1;
+----+------+-----+
| id | name | sex |
+----+------+-----+
| 1  | Yan1 | 1   |
+----+------+-----+
SELECT * FROM member_2;
+----+------+-----+
| id | name | sex |
+----+------+-----+
| 1  | Yan2 | 2   |
+----+------+-----+
SELECT * FROM member;
+----+------+-----+
| id | name | sex |
+----+------+-----+
| 1  | Yan1 | 1   |
| 1  | Yan2 | 2   |
+----+------+-----+

# 主表不能插入
INSERT INTO member(`name`, sex) VALUES('Yan3', 3)
> 1036 - Table 'member' is read only
```

> Error 1168 - Unable to open underlying table which is differently defined or of non-MyISAM type or doesn't exist.

报错原因：

- MERGE 引擎只适用于 MyISAM 表。
- UNION 了不存在的表。
- 各表的结构（索引、引擎、列、字符集等）不一致。

