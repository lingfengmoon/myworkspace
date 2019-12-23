# SQL语言

SQL语言是一个功能强大的数据库语言，用于与数据库的通信，SQL是关系型数据库管理系统的标准语言 。

一般分为：DDL（数据定义语言）；DDL用于定义数据的结构，如常见，修改，删除数据库对象

DML（数据库操作语言）；用于检索或修改数据。

DQL(数据库查询语言)；

DCL（数据控制语言，定义数据库用户的权限）；

TCL(事务操作语言)；

CCL（指针控制语言）；

## DDL（数据定义语言）

**DDL用于定义数据的结构，如常见，修改，删除数据库对象。建表，建库 ，主要是对数据库和数据库中表格的操作**

**** 关键字：CREATE  ALTER    DROP****

### 数据库的操作

```mysql
# 1、创建数据库
	create database my_first;
# 2、使用指定的字符集创建数据库
	create database my_second character set utf8;
# 3、查看系统中所有数据库
	show databases;
# 4、查看某一个数据库的创建信息
	show create database my_first;
# 5、修改数据库的默认字符集
	alter database my_second character set gbk;
# 6、删除数据库
	drop database my_second;
# 7、切换数据库
	use my_first;
# 8、查看当前正在使用的数据库
	select database();
```

### 表的操作

#### 表中数据类型

```mysql
表中的数据类型:
int: 整型，用来存储整数
double: 浮点型，用来存储小数。 double(4,3): 这个浮点型共4位，小数部分有3位。
char: 字符串，固定长度的字符串。 char(5): 固定占5位。
varchar: 字符串，可变长度的字符串。 varchar(5): 最多占5位，如果不到5位，则以实际占用为准。
text: 字符串，不限位数的。
blob: 字节类型。
date: 日期类型，格式是 yyyy-MM-dd
time: 时间类型，格式是 HH:mm:ss
timestamp: 时间类型，格式是 yyyy-MM-dd HH:mm:ss，会自动赋值
datetime: 时间类型，格式是 yyyy-MM-dd HH:mm:ss
```

#### 表的DDL语句

```mysql
# 1、建表(采用的字符集是数据库的字符集)
create table t_stu(tname varchar(20), tage int, tgendervarchar(6), theight int, tweight int);

# 2、查看当前数据库中所有的表
	show tables;
# 3、查看某一张表的创建信息
	show create table t_stu;
# 4、查看表的字段信息
	desc t_stu;
	
# 5、删除表
	drop table t_stu;
# 6、删除表中的某一列
	alter table t_stu drop tage;
	
# 7、添加列（新增列名字的后面必须加上类型）
	alter table t_stu add birthday date;
# 8、修改一个列的类型
	alter table t_stu modify theight double(5,2);
# 9、修改一个列的名字（修改列名，必须同时加上类型）
	alter table t_stu change theight height double(5,2);
# 10、修改表采用的字符集
	alter table t_stu character set gbk;
# 11、重命名表
	alter table t_stu rename t_user;
```

## DML（数据操作语言）

**用于检索或修改表格中的数据。 关键字：insert   delete   update**

```mysql
# 增加
#1 向表中所有列插入数据：
insert  into 表名values(列值1，列值2…列至N);
insert into t_user values ('小明', '2000-01-01', 'male', 172.5, 67, 98);
#1.1 向表中批量添加数
INSERT INTO user VALUES('小明','1996-12-2','male',120,45,80),
						('小王','2012-2-2','male',130,40,70),
						('小李','2005-1-8','male',160,70,80);

#2 向表中指定字段插入数据
insert  into 表名（列名1，列名2…列名N）values(列值1，列值2…列至N);
insert into t_user (`name`, `gender`, `birthday`) values ('小娟', 'female', '20000229');

#2.1 向表中指定字段批量添加数据
INSERT INTO `user` (`name`,gender,birthday)
				VALUES('小林','male','1992-02-03'),
					  ('小华','male','1995-12-03'),
					  ('小明','male','1998-02-08');
------------------------------------------------------------
#删除
#1 从表中删除数据（默认删除所有的数据；该删除表的结构还在可以找回）
	delete from t_user;
#2 从表中删除满足条件的数据
	delete  from 表名 where 条件；
	delete from t_user where name = '小明';
#3 删除表中所有的数据（不能找回，效率比delete高）
	truncate table 表名；
	truncate table t_user;
------------------------------------------------------------
#修改
#1 修改表中的字段对应的值（对所有的行生效）
	update t_user set score = 100, weight = 65;
#2 修改表中的字段对应的中（对指定的行生效）
	update t_user set height = 0 where height is null; 
	#Set后面跟的都是 =null;
```

## DQL(数据库查询语言)

非常重要的执行操作,这个操作不会对数据进行改变,而是让数据查询后的结果返回给客户端，将查询的结构生成一张虚拟表,展示数据 

关键字：select *：代表所有列

select   列名 --> 要查找的列
from   表名 -->列是在那张表中的
where   条件语句 -->以满足什么条件来查询
group by  --> 对查询的结果进行分组
Having   条件语句 -->对分组后的查询条件 (对分组过后组内部使用条件)
order by   --> 对查询结果进行排序
limit   -->限定结果 

写列名的时候可以使用反引号(``)。作用：如果列名和关键字重复，可以使用该方式表示这是一个列名，不重名不需要

### 书写顺序 ：

**select --> from --> where --> group by --> having --> order by -->limit** 

### 执行顺序：

**from --> where --> group by --> having --> select ---> order by --->limit** 

**where后面不能用聚合函数**

### 运算符优先级问题：

（）的优先级最高，其次关系运算符（>; <; >=; <=;  =; != ）最后是逻辑运算符（&&；||；！）



```mysql
查询表中所有数据：
	select * from 表名；
```

### 避免出现重复数据的查询：

**distinct**(（去除列的数据必须完全相同）)

```mysql
distinct：
语法：select distinct 列名1，列名N  from 表：可以去除多余的重复

e.g:查询dir_id 和salePrice 两列（除去重复部分）
SELECT DISTINCT dir_id ,salePrice FROM product;

```

### 数学运算符查询：

实现数学运算符查询:数据类型可以使用算术运算符（+ - * /）；对date数据类型部分可以使用+ -

```mysql
/*
1 先乘除，在加减
2 同级顺序从左到右
3 表达式中可以使用（）改变计算优先级
*/
# as ：别名，主要用于展示可以用于后续计算，as可以省略，就近处理原则，若别名中使用特殊字符空格等需要加单引号。
e.g #查询所有货品ID，名称和批发价格(批发=零售*折扣)（totalCostPrice:别名，以该名字的形式展现）
SELECT id,productName,salePrice*cutoff FROM  as totalCostPrice product;
```

### 设置自定义显示查询：

**concat** :拼接字符串

```mysql
/*mysql中提供内置函数,这个函数可以提供处理数据的方式:concat
*/
e.g #查询商品的名字和零售价格:格式: XXX商品的零售价格为:XXX
select CONCAT(productName,'商品的零售价格:',salePrice) as productSalePrice from product; 

```

### 设置别名

**AS**：就近处理原则：as可以省略不写，离哪个字段近就变哪个

```mysql
/*
1 在order by排序中 别名不能使用中文
2 别名中如果有特殊符号需要加单引号
3 别名可以做前列条件进行计算
*/
# 查询所有货品id ，名称，各进50个，并且每个运费成本1元
SELECT id,productName,(costPrice+1)*50  AS totalcostprice FROM product;
SELECT id,productName,(costPrice+1)*50 totalcostprice FROM product;#该句省略了别名
```

### 过滤查询：

关系运算符：**= , >  ,< ,  >=,  <=, !=(<>)**

SQL各子句的还行先后顺序
1.from子句,先确定从哪一张表中去做查询
2.where子句,从表中直接筛选出符合条件的数据
3.select子句,从筛选之后的结果集中现实出某些列

4 order by: 对查询结果进行排序

```mysql
/*
语法：select 列名(*代表所有列) from 表 where 子句 返回限定结果;
where作用：限制查询结果，根据where后面的条件决定查询结果
*/
e.g#查询商品价格为119的所有商品
SELECT * FROM product WHERE salePrice =119;
#查询分类编号不是2的货品信息
SELECT * FROM product WHERE id<> 2;
#查询货品名称,零售价格小于等于200的货品
select productName,salePrice from product where salePrice <= 200;
# 查询id,货品名称,批发价格大于350的商品(要求使用别名)
SELECT id,productName,salePrice*cutoff as 批发价格 FROM product WHERE salePrice*cutoff>350;# as 就近原则 --> 取后面的名字 ---> 省略as关键字
```





### 逻辑运算符查询：

**AND（&&）；OR（||）；NOT（！）**

```mysql
#查询 id,货品名称,批发价格在300~400之间的所有货品
SELECT id,productName,salePrice*cutoff FROM product WHERE salePrice*cutoff >= 300 AND salePrice*cutoff <=400;

```

### 范围查询：

**between and;******(between  A and B :在AB之间包括AB)****

```mysql
/*
语法：select 列名 from 表名 where 列名 between 最小值 and 最大值;
*/
e.g #查询 id,货品名称,批发价格在300~400之间的所有货品
SELECT id, productName,salePrice*cutoff FROM product WHERE salePrice*cutoff BETWEEN 300 and 400;
```

### IN运算符（判断是否在指定集合）：

```mysql
/*判断列的值是否在指定的集合(范围)内
可以用于字符型范围判断
语法：select 列名 from 表名 where 列名 IN(值1,值2....);
*/
e.g # 查询 id,货品名称,分类编号不为2,4的所有货品
select id,productName,dir_id from product where dir_id != 2 AND dir_id != 4;
select id,productName,dir_id from product where NOT (dir_id =2 OR dir_id = 4);
select id,productName,dir_id from product where NOT dir_id IN(2,4);
```

### 空值查询：

**IS NULL** 数据库中允许列中没有值（数据），无数据时使用null作为值存储。

```mysql
/*判断列的值是否为null，可以使用空值查询 IS NULL
不为空：is not null;
语法：select 列名 form 表名 where 列名 IS NULL; 
*/
e.g #查询商品名称为NULL所有商品
select * from product where productName IS NULL;
```

### 模糊查询：

**LIKE :** 进行查询条件模糊匹配。

 **%（通配符）：表示0个或者多个任意字符；**

**—（下划线，通配符）：表示一个字符**

%在查询条件前面表示以xxx开头，%在查询条件的前面表示以xxx结尾；

%和 — 可以同时出现在匹配条件中;

```mysql
/*
语法：select 列名 from 表名 where 列名 LIKE 模糊查询条件；
*/
e.g #查询 id,货品名称,货品的名称需要匹配 '罗技MXXXX'所有的货品
select id,productName from product where productName LIKE '罗技M%';
#查询 id,货品名称,分类编号,零售价格大于等于200的货品或者匹配'罗技M1'后两字符
select id,productName,dir_id salePrice from product where salePrice>=200 OR
productName LIKE '%罗技M1__';
```

### 结果排序：

**ORDER BY:**对查询结果进行排序 ；ASC：升序，不写默认升序；DESC:降序

ps: ORDER BY不能给中文排序，所以这里别名不能使用中文

```mysql
/*
语法：select 列名 from 表名 where 条件 order by 列名 [ASC/DESC] ....;（可以对多个列名进行排序）;
1 如果语句中无where,ORDER BY就跟在from后面书写
2 order by子句是在select子句后面执行，因此uselect中使用的别名可以在 order by中进行使用.
3 需要排序的别名不能使用中文
*/
e.g #查询 id,货品名称,分类编号,零售价格按分类编号排序(降序),在按照零售价格排序(升序)
#多行的情况下是根据列值相同进行排序的-->参考 comparable 接口的实现
select id,productName,dir_id,salePrice from product order by dir_id DESC,salePrice ASC;
#查询M系列并按照批发价格排序(降序)
select id,productName,salePrice*cutoff as pf from product where productName LIKE '%M%' order by pf DESC;
```

###  分页查询/限制提交：

**limit: 限制提交结果**

```mysql
/*
limit m，n : 表示从m条开始查到n条数据展示出来；
limit n :    表示从0到 n条（查询的总条数为n条）
limit (m-1)*n,n : 表示查询第 m 页的数据，每页大小为 n;
*/
e.g #查询分类为2并按照批发价格排序(升序)
select id,productName,salePrice*cutoff as pf from product where dir_id = 2 order by pf limit 3
```

### 统计函数(聚合函数)：

**count; max; min; sum; avg**

**所有的聚合函数，都会忽略字段为NULL的那条记录**

**count(*)** **不会忽略null值所在的行记录，即总行数**

**聚合函数里面的参数只能有一个**

count --> 统计结果次数
max ---> 统计最大值
min ---> 统计最小值
sum ---> 求和
avg ---> 平均值

**if null (expr1,expr2)**:如果expr1的值为null就使用默认值expr2;如果不为null 就使用它自己本身的值

```mysql
#查询求出所有商品平均零售价格
select avg(salePrice) from product;
#统计商品的个数 一般这个列统计的是主键列即 id列 (count这个值在java中必须使用long接收)
select count(id) from product;
#统计商品的个数,分类为2的商品
select count(id) from product where dir_id = 2;
#查询商品零售价格的最小值,最大值,以及所有的零售价格总和
select min(salePrice),max(salePrice),sum(salePrice) from product;
```

### 分组查询：

**group by ; having;**

**分组查询中，在select的子句中只能放分组的字段**；**注意考虑空值问题**

where 和having的区别：

1.**where是分组前对数据进行过滤**； 

 2.**having是分组之后对数据进行过滤** 。having后面可以使用分组函数(统计(聚合)函数) where后面不可以使用分组函数 。

3  where是对分组前的条件,如果某行记录没有满足where子句的条件,那么这行记录不会参加分组而having是对分组后数据的约束 


```mysql
/*
分组查询
*/
#查询每个部门的部门编号和每个部门的工资和
select deptno,sum(sal) from emp group by deptno;
#having子句; 主要作用是作用在分组之后对数据进行过滤
#查询工资总和大于9000的部门编号以及工资
select deptno,sum(sal) from emp group by deptno having sum(sal) > 9000;
```

## 表的约束：

表的完整性作用：**保证用户输入数据准确的保存到数据库中**；

为了保证表的完整性，我们可以为表添加约束（实体完整性，域完整性，引用完整性）；通常在进行DML操作时，必须符合约束条件，否则不能执行

### 实体完整性：

**作用：标识每一行数据不重复；**

可以使用的约束类型：

**主键约束primary key：非空（null）且唯一**

**唯一约束 unique  ：   null不算重复**

**自增长列auto_incrment**  :   

```mysql
/*主键约束 primary key;
ps:每个表中需要有一个主键，非空（null）且唯一
*/
#第一种添加方式：主键约束primary key
create table student(
	id int primary key,#添加主键进行主键约束
    name varchar(50)
);
-----------------------------------------------------
#第二种添加方式：此方式多用于联合主键。一个表中若有多个主键，主键名称用逗号隔开
create table student(
id int,
name varchar(50),
primary key(id)#添加主键进行主键约束
);
-----------------------------------------------------
#第三种添加方式:
create table student(
id int,
name varchar(50)
);
alter table student add primary key(id);#忘记添加主键之后通过alter加上

```

```mysql
/*唯一约束unique（null 不算重复）
特点：不能重复，约束存储数据必须是唯一的
*/
create table student(
	id int primary key,
    name varchar(50) unique #通过unique进行唯一约束，该列不能重复，存储数据必须唯一
	);
```

```mysql
/*自动增长列
特点：自动增长列往往是整数值，自动增长的列不需要赋值，如果需要赋值指定一个值（null或0）
*/
create table person(
	id int primary key auto_increment,#通过auto_increment约束可以实现自动增长。
	name varchar(50)
	);
	
	#以下插入数据语句会实现id自动增长
	insert into person values(null,'ss'),
							 (null,'sss'),
							 (null,'ssss'),
```

### 域完整性：

**作用：限制此单员格内部的数据的正确性**

**非空约束：not null ;   默认约束： default;**

```mysql
/*非空约束 
特点：所约束的列存储的内容不能为null
*/
create table student(
	id int primary key,
	name varchar(50) not null,#通过not null来约束name不能为空
	gender varchar(10)
	);
-----------------------------------------------------
#建表后限制非空约束：
alter table student modify name varchar(50) not null;
```

```mysql
/*默认约束
defaule 
*/
create table student(
	id int PRIMARY KEY,
	name varchar(50) not null ,
	gender varchar(10) default '男'#默认该列不写为男
);
INSERT INTO student(id,name,gender)
				values(1,'tom','女');
#对表种每行类都添加数据,此时可以省略列名
INSERT INTO student values(2,'jerry',default);#默认为‘男’
```



### 引用完整性：

多用于多表查询，  **外键约束：foreign key**

```mysql
/*外键约束
references ：连接
*/
create table student(
	sid int primary key,
	name varchar(50) not null,
	gender varchar(10)default '女'
	);
create table score(
	id int ,
	score int,
    sid int # 外键列的数据类型必须和主键列的数据类型一致
	constraint fk_sc_stu foreign key(sid) references student(sid) #student中的主键 sid 作为score中的外键
	);
	# fk_sc_st：起的约束名字；表示该列由sc表中的外键，st表中的主键。
```

## 多表查询

### 合并结果集查询

union:两个查询结果有相同数据，去除重复值

union all:查询结果不去除重复值

```mysql
/*多表合并结果的查询
union:去除重复值；union all:不去除重复值
两个查询语句使用者俩关键字连接即可，查询语句的字段名，个数必须对应上
*/
create table t1(
	id int,
	name varchar(5)
	);
INSERT INTO t1 values(1,'a');
INSERT INTO t1 values(2,'b');
INSERT INTO t1 values(3,'c');
INSERT INTO t1 values(4,'d');

create table t2(
id int,
name varchar(5)
    );
INSERT INTO t2 values(4,'d');
INSERT INTO t2 values(5,'e');
INSERT INTO t2 values(6,'f');
#去除t1表和t2表查询结果的重复值
select * from t1 UNION select * from t2;
#合并但是不去重复
select * from t1 union all select * from t2;
```

### 连接查询

做多表联合查询的时候，一定要避免产生**笛卡尔积**

**笛卡尔积**：通过SQL语句进行多表联合查询时候，没有对条件进行限制,此时会出现A表中的每条数据和B表中每条数据产生关联,这个关联其中有些数据是无用数据(笛卡尔积),那么我们在查询的时候就需要避免出现这些无用数据 

**避免笛卡尔积的方式就是处理外键和主键之间关系** 

```mysql
/*内连接：   表1 inner join 表2 On 条件
两表合并根据条件产生需要的数据
*/
#方言版内连接
select e.ename,e.sal,e.comm,d.dname from emp e,dept d where e.deptno = d.deptno;
#标准sql内连接
select * from emp e inner join dept d ON e.deptno = d.deptno;
```

```mysql
/*左外连接;  表A left outer join 表B on 条件
 左表为基础表，右表为辅表；以满足左表的数据为先决条件，若右表没有左表的数据，以null显示
*/
#emp为左表,dept为右表，以左表为基础连接两张表，根据条件查出数据
select * from emp left outer join dept d ON e.deptno = d.deptno;
```

```mysql
/*右外连接 表A right outer join 表B on 条件
右表为基础表，左表为辅表；以满足右表的数据为先决条件，若左表没有右表的数据，以null显示
*/
#emp为右表,dept为左表，以右表为基础连接两张表，根据条件查出数据
select * from emp e right outer join dept d ON e.deptno = d.deptno;
```

### 子查询

```mysql
/* 1 在where后面,作为查询条件的一部分
*/
#查询工资大于员工编号为7369这个员工的所有员工信息
select * from emp where sal>(select sal from emp where empno = 7369);
#查询工资大于10号部门的平均工资的所有员工信息
select * from emp where sal>(select avg(ifnull(sal,0)) from emp where deptno=10);
# 查询工资大于10号部门的平均工资的非10号部门的员工信息。
select * from emp where sal>(select avg(ifnull(sal,0)) from emp where deptno=10) and deptno<>10;
# 查询与7369同部门的同事信息。
select * from emp where deptno=(select deptno from emp where empno=7369) and empno<>7369;
------------------------------------------------------------
/*2 在from后面,作为一个表存在；使用表别名
*/
# 查询员工的姓名，工资，及其部门的平均工资。
elect A.ename,A.sal,B.avg_sal from emp A join (select deptno,avg(ifnull(sal,0)) avg_sal from emp group by deptno) B on A.deptno = B.deptno
------------------------------------------------------------
/*3 在having子句中
*/
# 查询平均工资大于30号部门的平均工资的部门号，和平均工资 
select deptno,avg(ifnull(sal,0)) from emp group by deptno having avg(ifnull(sal,0))>
(select avg(ifnull(sal,0)) from emp where deptno=30);
------------------------------------------------------------
/*4 在select子句中
*/
# 查询每个员工的信息及其部门的平均工资，工资之和，部门人数

```

```mysql
/*子查询:嵌套查询，
子查询可以出现的位置
1.可以出现在where后面,作为查询条件的一部分
ps:出现在wehere后面可以使用关键字 any / all
2.可以出现在from后面,作为一个表存在；使用表别名
3.作为列存在
*/
#查询 工资高于部门编号30的所有员工信息
#第一步:查询出来部门编号30的员工中的最高薪水
select max(sal) from emp where deptno = 30;
#第二步:查询整个部门高于部门编号30的员工信息
select * from emp where sal > '第一步信息'
#查询:
select * from emp where sal > (select max(sal) from emp where deptno = 30);
#使用一个关键字 all 使用在where子句的后面 代表所有情况(取出子表中所有的数据)
select * from emp where sal > ALL(select sal from emp where deptno = 30);
```