# dHive

## 库操作

```mysql
1 #建库操作：
	create database [if not exists] 库名称 [comment （数据库注释）];

2 #查询数据库信息
	desc database [extended（显示详细信息）] 库名称;

3 #删除库（没有表的普通删除）
	drop database 库名；
  3.1 #强制删除（库下的表都会被删除）
    drop databse 库名 cascade;
```

## 表操作

```mysql
1 #建表操作
    create [external] table [if not exists] 表名[库名.表名](
    列名 数据类型, .....
     )[comment '注释'];        
row format delimited fields terminated by '指定读取分隔符' ;
 
2 #查看所有表名
    show tables;
   #查看表结构
    desc [extended] 表名称    
3 #修改表名
    alter table 原表名 rename to 新表名；
   #修改列名（列类型）
    alter table 表名 change column 原列名 新列名 列类型；
   #修改列的位置（将列1修改到列2之后）
 alter table 表名 change column 列名1  列名类型 after 列名2 ；  
   #修改列至最前面
    alter table 表名 change column 列名1  列名类型 first;    
4 #增加字段
	alter table 表名 add columns (列名 类型，...);
5 #删除表
	drop table 表名
------------------------------------------------------------
# 向表中加载数据
 1 # 本地加载数据
	load data local inpath '本地存储数据的路径' [overwrite] into table hive库中表的名字
 2 # HDFS加载数据
	load data inpath 'HDFS数据路径' [overwrite] into table hive库中表的名字
 3 # 动态加载数据
	insert into table tableName2 select [.....] from tableName1;
-----------------------
# 克隆表
 1 # 只克隆表的结构
	create table if not exists 新表名 like 要克隆的表
 2 # 克隆表带数据
 	create table if not exists 新表名 like 要克隆的表 location '存储数据的目录'
 3 # 克隆表简便方式（Hive独有）
	create table if not exists 新表名 as select 列名 from 克隆表的名字 where 条件；
```

## 数据类型

### 基本数据类型

```mysql
 tinyint(1); smalint(2); int(4) ; bigint(8); boolean;  
 float;  string;  binary(字节数组)；
 timestamp(时间戳)； date(日期)；
```

### 复杂数据类型

```mysql
# 将复杂数据类型展开
	explode ()
# 虚拟表
	lateral view 
# 拼接字符串
 	concat(string，'以什么拼接'，string);
 	concat(str1,str2)
# 将多行数据搜集起来变成一个数组
	concat_set() 
#将多个字符串连接成一个字符串，但是可以一次性指定分隔符
	concat_ws('字符串之间分隔符'，str1,str2) 
# 将字符串拼成键值对
	str_to_map ('字符串','键值对之间的分隔符'，'k和V之间的分隔符') 
# 数据类型强转 （将某列的数据类型进行强转）
	cast(列名 as 数据类型)
```



#### Array

```mysql
#建表：
crate table if not exists 表名(
				列名 列的数据类型,
				........
				列名 array<数据类型>
					)
row format delimited fields terminated by '数据分隔符'
collection items terminated by '特殊数据的分隔符'
----------------------------
# 案例：
数据：
shangsan 78,89,92,96
lisi 67,75,83,94
建表：
create table if not exists arr1(
        name string,
        score array<String>
        )
row format delimited fields terminated by '\t'
collection items terminated by ',';
加载数据：
load data local inpath '/root/arr1.txt' into table arr1;

查询
select * from arr1;
  # 获取一个元素使用下标
select name,score[1] from arr1;
 # 下标越界不会出现报错 而是返回NULL值
select name,score[10] from arr1;
 # 获取数据的长度
select size(score) from arr1;
# 限制取值问题
select score[10] from arr1 where size(score)>=10;
--------------------------
需求：将score展示成下列样式
zhangsan 78
zhangsan 89
zhangsan 92
zhangsan 96
/** 展开数组
  explode 展开(复查数据类型)
  lateral view :虚拟表
  concat_ws :多行变一行
  ps:虚拟表与explode连用可以将数据展开并结合前面属性展示
*/
 	# 1 将成绩展开
select explode(score) from arr1; 
  # 2 求出每个学生的总成绩（explode和lateral view 联合查询）
select name,sum(sc) from arr1 lateral view explode(score) score as sc group by name;


先执行 from arr1表中的score这一列展开形成一张虚拟表,表中有一类score 会将展开这些一列的数据存储到这张虚拟表中,给这张虚拟表的score类 起了一个别名,sc select 使用这个sc 代表虚拟表中score这一列的数据
ps:虚拟表会和展开 一起使用形成数据表 多用于复杂数据类型
虚拟表的生命周期 仅限查询范围内
 求每个学生总成绩结果:
	lisi 319.0
	shangsan 355.0

# 3 将虚拟表中数据转换到一个实体表(相当于虚拟表数据插入到arr_temp中)
create table arr_temp as select name,sc from arr1 lateral view explode(score) score as sc;

数据: 表 arr_temp 就是下列这个样式
shangsan 78
shangsan 89
shangsan 92
shangsan 96

需求： 将上列数据样式转成下列样式:
      shangsan [78,89,92,96]、
---------------------------------------     
/*合并数组
 collect_set（） : 将数据合并
*/
   # 1 创建存储数据的表
    create table if not exists arr2(
            name string,
            score array<string>
            )
    row format delimited fields terminated by ' '
    collection items terminated by ',';

 # 2  将分散数据合并（合并插入一个数组格式）
insert into arr2 select name,collect_set(sc) from arr_temp group by name;

ps:拆分过程就是行转列，合并过程就是列转行
```

#### Map

```mysql
1 #建表格式：
create table if not exists 表名(
        列名 列的数据类型,
        ........
        列名 map<数据类型,数据类型>
			)
row format delimited fields terminated by '数据分隔符'
collection items terminated by '数组的分隔符'
map keys terminated by 'map的分隔符'
----------------------------------------
# 案例数据:
zhangsan chinese:90,math:87,english:63,nature:76
lisi chinese:60,math:30,english:78,nature:0
wangwu chinese:89,math:25,english:81,nature:9

ps:将Map数据看做是数组中存储了kv键值对 ,需要先拆分数据在拆kv对

#建表:
	create table if not exists map1(
            name string,
            score map<string,int>
            )
    row format delimited fields terminated by ' '
    collection items terminated by ','
    map keys terminated by ':';
#加载数据:
load data local inpath '/root/map1.txt' into table map1;

# 1 查询：
    # 搜索全部
	select * from map1;
	
   # 取出某一个key对应的value值
   #格式： map的列的名字 score[字符串key的名字]
 	 select name,score['chinese'] from map1;
  
   # 传入一个不存在的key会返回NULL
	select name,score['XXXXX'] from map1;
	
  # 查询数学大于35分的升学生的英语和自然成绩
	select name,score['english'],score['nature'] from map1 		where score['math']>35
	;
  # ps:可以使用别名的方式处理 建议在from写后面写 因为优先级最高
select m.name,m.score['english'],m.score['nature'] from map1 m where m.score['math']>35;
---------------------------------------------
2 /** 展开
  explode 展开(复查数据类型)
  lateral view :虚拟表
  ps:虚拟表与explode连用可以将数据展开并结合前面属性展示
*/ 
    # 展开map1表中的score   ----->explode(要展开的列);
   # ps:因为是map即键值对所以列要有两个
select explode(score) as (className,classScore) from map1;
  
  # 将展开的列以及其他列都显示
  # 使用虚拟表(lateral view)和展开(explode)结合查询
select name,className,classScore from map2 lateral view explode(score) score as className,classScore;
--------------------------------------------
3 /**合并
str_to_map函数 将字符串进行键值对拼接
concat 拼接字符串
*/

  # 3.1 创建map_temp表存储虚拟表中数据（也就是克隆表）
create table map_temp as select name,className,classScore from map1 lateral view
explode(score) score as className,classScore;
 # 3.2 将多行数据搜集变成一个数组  collect_set
select name,collect_set(concat(classname,":",classscore)) 
from map_temp group by name;
 #3.3 将多个字符串合并成一个字符串   concat_ws
 select name,
 concat_ws(",",collect_set(concat(classname,":",classscore)）)  from map_tempgroup by name;
  # 3.4 将字符串转成map键值对  str_to_map
 select name,
str_to_map(concat_ws(",",collect_set(concat(classname,":",classscore))),',',':')
from map_temp group by name;          

```

#### struct（结构体）

```mysql
# 结构体是一个特殊类(class) 只能存数据不能存方法
# 建表格式
create table if not exists 表名(
        列名 列的数据类型,
        ........
        列名 struct<数据名:数据类型,数据名:数据类型.....>
)
row format delimited fields terminated by '数据分隔符'
   # ps:根据数据的不同添加不同方式
collection items terminated by '数组的分隔符'
map keys terminated by 'map的分隔符'
------------------------------------
#案例 建表：
create table if not exists str1(
name string,
score struct<chinese:int,math:int,english:int,tiyu:int>
)
row format delimited fields terminated by '\t'
collection items terminated by ','

# 加载值:
load data local inpath '/root/arr1.txt' into table str1;

ps:结构体也要有<>, 结构数据查询出来之后发现和map的展示数据是一样
但是需要注意,结构体中属性是我们自己写,map中key是读取数据中读取进来

获取结构中数
select name,score.chinese from str1;
```

## 函数

### 内置函数

```mysql
ps:函数时可以直接参与查询
1.#随机函数 rand(int seed)
直接使用 rand() --> select rand() 范围是0~1之间的double值
范围内随机 rand()*随机范围 --> select rand()*10 范围内稳定随机序列

2.#切分 split(字符串是什么,切分的符号);会的会得到一个数组
select split('a,b,c,d',','); --> 返回值 ["a","b","c","d"]
select split('a,b,c,d',',')[0]; --> 获取一个字符

3.#字符串截取 substr,substring都是一样的函数
substr(字符串,什么位置开始,到什么位置结束(相当于是长度))
substring(字符串,什么位置开始,到什么位置结束(相当于是长度))
select substring('abcdefg',0,2); --> 'ab'

4.#正则表达式 regexp_replace
regexp_replace(字符串,正则条件(匹配的条件),替换成什么(字符串))
select regexp_replace('1.jsp','jsp','html'); --> 1.html

5.#数据转换 cast(值 as 数据类型)
select cast("12" as int); --> 转换为12是int类型
如果转换失败是 NULL填充

6.#支持分支语句
    6.1 #if函数
        if(判断条件,为true返回的值,为false返回的值)
        select if(100>10,'this is true','this if false');
    6.2 # case when
        格式:
        case c值
        when w值 then 结果
        ........
        else 结果
        end;
先计算c值然后和w值进行匹配若匹配成功执行then后面值返回,如果不成功会继续下下匹配,当执行到else就证明已经到了最后一个结果即所有的when都没有匹配成功 end 是结束符
select
case 6
when 1 then '100'
when 2 then '200'
else 'others'
end;

案例：
1.#准备数据和建表
name dept_id sex
兰兰 A 男
昊哥 A 男
绿绿 B 男
华华 A 女
秀秀 B 女
婷婷 B 女
建表
create table if not exists emp_sex(
name string,
dept_id string,
sex string
)row format delimited fields terminated by '\t';
#加载数据
load data local inpath '/root/emp_sex.txt' into table emp_sex;
2．需求
# 求出不同部门男女各多少人。结果如下：
A 2 1
B 1 2
select
dept_id,
sum(case sex when '男' then 1 else 0 end),
sum(case sex when '女' then 1 else 0 end)
from emp_sex
group by dept_id;

7.行转列
   7.1 # 字符串A拼接符号字符串B
       concat 字符串和拼接concat(字符串A,拼接符号,字符串B) 
    7.2 #字符串A|字符串B|字符串C|......
   	   concat_ws(拼接符号,多个字符串....)
ps:如果拼接符号是一个NULL,也会使用NULL拼接(拼接方式相当于给空字符串) 
	7.3 #拼接返回一个数组(排除重复)
	   collect_set(只支持基本数据类型) 
#案例:
name constellation blood_type
华华     白羊座        A
婷婷     射手座        A
昊哥     白羊座        B
秀秀     白羊座        A
绿绿     射手座        A

#需求:
把星座和血型一样的人归类到一起。结果如下：
射手座,A 婷婷|绿绿
白羊座,A 华华|秀秀
白羊座,B 昊哥

#创建表加载和数据
create table if not exists per_info(
name string,
constellation string,
blood string
)row format delimited fields terminated by '\t';
load data local inpath '/root/constellation.txt' into table per_info;

# 实现语句：
select
t1.cb,concat_ws('|',collect_set(t1.name))
from
(select name,concat(constellation,',',blood) as cb from per_info) t1 group by t1.cb;

8 列转行
   #展开 --> 复杂数据类型数据进行拆分成多行
	explode 
   #虚拟表 -->通过展开可以的到 一张虚拟表,再次操作虚拟表都可以得到数据
   Lateral view  
 
#案例：
建表加载数据
create table if not exists movie_info(
name string,
category array<string>
)row format delimited fields terminated by '\t'
collection items terminated by ',';
load data local inpath '/root/movie.txt' into table movie_info;

# 需求：展开数据
select name,category_name from
movie_info lateral view explode(category) ex as  category_name;

ps：行转列还是列转行都是比较常见第一种数据转换方式,主要的目的就是讲已经有序的数据,转换一种存储形式,可以更加方便对数据操作
----------------------------------------------
字符串函数：
lower--（转小写）
upper--（转大写）
length--（字符串长度，字符数）
concat--（字符串拼接）
concat_ws # 数值变字符串
substring(str from pos for len)
-----------------------------------------
数学函数
#四舍五入((42.3 =>42))
	round （）
#向上取整(42.3 =>43)
	ceil 
#向下取整(42.3 =>42)
	floor 
```

### 日期函数

```mysql
# 获取当前时间 
select current_date from dual;  -- 2019-03-15
select current_timestamp from dual; --2019-03-15 15:47:25

# 查询当月第几天 
select dayofmonth(current_date);
#月末:
select  last_day(current_date)
#当月第1天: 
select date_sub(current_date,dayofmonth(current_date)-1)
#下个月第1天:
select  add_months(date_sub(current_date,dayofmonth(current_date)-1),1)
 
# 时间戳转日期
select from_unixtime(1505456567); 
select from_unixtime(1505456567,'yyyyMMdd'); 
select from_unixtime(1505456567,'yyyy-MM-dd HH:mm:ss'); 
# 日期转时间戳
select unix_timestamp('2017-09-15 14:23:00'); 

# 20190820 --->2019-08-20
from_unixtime(unix_timestamp('20190820','yyyyMMdd'),"yyyy-MM-dd")

#计算时间差
select datediff('2018-06-18','2018-11-21');

#字符串转时间（字符串必须为：yyyy-MM-dd格式）
select to_date('2017-01-01 12:12:12');

#日期、时间戳、字符串类型格式化输出标准时间格式：
select date_format(current_timestamp(),'yyyy-MM-dd HH:mm:ss');
select date_format(current_date(),'yyyyMMdd ');
select date_format('2017-01-01','yyyy-MM-dd ');  
```



### 窗口函数 

```mysql
over(如何划分窗口):指定分区函数 工作的范围是数据窗口大小,这个数据窗口大小可能会随着行的变化而变化
over其实就是我们认知的分区,根据规则向相同数据放到一个窗口(想象成分区内部)内部,对窗口内部的数据进行操作 
over开窗函数的本质就是增加了一个语法,这个语法可以根据分区原则对窗口内即分区内的数据进行操作

/**可以写在over函数中的属性:
current row :当前行
n preceding : (n行数) 往前n行数据
n following : (n行数) 往后n行数据
unbounded: 起点状态
unbounded pereceding : 起点，表示从前面的起点
unbounded pereceding : 终点,表示到后面的终点
lag(所要查询的列,n,默认值):往前第n行数据  --默认值可以没有
lead(所要查询的列,n,默认值):往后第n行数据
ntile(n):把有序分区中的行发送到指定的组中(n是有几组),各个组有编号,编号是从1开始(n必须是int类型) 
partition by -->分区 分区排序必须使用 order by
distribute by  ----> 分区 分区排序必须使用 sort by 
*/

#案例:
数据：
name  orderdate  cost
jack 2017-01-01  10
tony 2017-01-02  15
iack 2017-02-03  23
marr 2017-04-12  30
tony 2017-01-04  29
jack 2017-01-05  46
tony 2017-01-08  55
marr 2017-05-10  12
marr 2017-06-12  80

建表: 
create table if not exists business(
name string,
orderdate string,
cost int
)
row format delimited fields terminated by ',';
加载数据:
load data local inpath '/root/business.txt' into table business;

#查询在2017年4月份购买过的顾客总人数
select name,count(*) from business
where substring(orderdate,1,7)='2017-04'
group by name;

ps:substring,substr有一个参数不太严谨,第一个字符的位置应该无论是1还是0都是第一个字符
over函数可以跟在聚合函数后面,对聚合内部的数据进行计算
over什么都不写,就相当好是一个窗口

#查询顾客的购买明细以及月购买的总额
select name ,orderdate,cost,sum(cost) over(partition by month(orderdate)) from business;
#相当于下面这种写法：
select name,orderdate,cost,sum(cost) from business
group by month(orderdate);
ps:根据月份进行分区是一个窗口中,统一一次记过

#将每一个顾客的cost按照日期进行累加
  # 累加所有行的结果
select name,orderdate,cost,sum(cost) over()from business;

  #按照name进行组内数据累加
select name,orderdate,cost,sum(cost) over(partition by name)
from business;

  #累加完排序(组内数逐行累加)
select name,orderdate,cost,sum(cost) over(partition by name order by orderdate)from business;

 # 做组内聚合(组内数逐行累加排序)
select name,orderdate,cost,sum(cost) over(partition by name order by orderdate rows between unbounded preceding and current row)  from business;
ps:rows必须跟在order by 子句的后面,对排序的结果进行限制,使用固定行数来限制分区中的数行数量

    # 扩展语句:
    # 当前行和前面一行做聚合
    select name,orderdate,cost, sum(cost) over(partition by 	name order by orderdate
    rows between 1 PRECEDING and current row) from business;

    # 当前行和前边一行及后面一行
    select name,orderdate,cost,
    sum(cost) over(partition by name order by orderdate 
    rows between 1 PRECEDING AND 1 FOLLOWING)
    from business;

	# 当前行及后面所有行 (都是在同一个分区的内部)
    select name,orderdate,cost,
    sum(cost) over(partition by name order by orderdate
    rows between current row and UNBOUNDED FOLLOWING)
    from business;

#查看顾客上次购买的时间
select name,orderdate,cost,lag(orderdate,1,'1900-01-01') over(partition by name order by orderdate) from business;
解析：lag(orderdate,1,'1900-01-01') 
查找orderdate的上一行即上一次的购买时间，如果没有返回默认值1900-01-01

#查询前20%时间的的订单信息
select * from (select name,orderdate,cost,ntile(5) over(order by orderdate) as sortedfrom business) t
where sorted = 1
```



### 排名函数

```mysql
/**
row_number() 从1开始按照顺序,生成分组内记录的顺序
ps:row_number的值不会存在重复,当排序的值相同的时候,按照表中记录的顺序进行排列
例如: 1 2 3 4 5.....

rank() 生成数据项在分组中排名,排名相等会在名词中留下空位
ps:两个第2名,下一名不会是3而是4
例如: 1 2 2 4 5 6.....

dense_rank() 生成数据项在分组中排名,排名相等会在名次中不留下空位
ps:两名第2,下一名是3
例如: 1 2 2 3 4 5 ....
使用over对函数进行进行一定的限制
*/

# 案例:
#建表 数据（id，班级，分数）
create table if not exists rstu(
id int,
class string,
score int
)row format delimited fields terminated by ' ';
load data local inpath '/root/rstu.txt' into table rstu;

# 按照班级的成绩进行排序(降序)
select *,row_number() over(distribute by class sort by score desc) from rstu;

# 取出班级的前三名(topN)
select a.*
from(select *,row_number() over(distribute by class sort by score desc) rm from rstu) a where a.rm < 4;

#展示所有排名函数
select *,
row_number() over(distribute by class sort by score desc),
rank() over(distribute by class sort by score desc),
dense_rank() over(distribute by class sort by score desc)
from rstu;
```



### 自定义函数

自定义函数分类：UDF:一对一的输入输出；UD

​				

1 需要创建一个类,这个类需要继承于UDF
2 重写evaluate() -->函数的处理逻辑写在这个方法中 

#### 案例1 :字符串拼接

```java
//编写自定义函数逻辑-->java类
1 // 第一个UDF字符串拼接
import org.apache.hadoop.hive.ql.exec.UDF;
import com.google.common.base.Strings;
public class FirstUDF extends UDF{
public String evaluate(String str) {
//这是一个工具类 Strings字符串专门用来判断是NUll 还是""
if(Strings.isNullOrEmpty(str)) {
return null;
} 
    return str+"_class";
}
}
```

3 将自定义UDF函数上传到Hive中使用
ps:无论那种都需要将程序打成 jar 

```mysql
# 第一种：
1、将编写好的udf的jar包上传到服务器，并添加到hive的class path中：
在hive中输入 add jar /root/jar包名.jar;
2、创建一个临时的自定义函数名
create temporary function myfunc as '类的全限定名';
3、查询是否创建成功。
show functions;
4、测试函数功能。
小技巧
dual这个库的名字是 oracle数据库提供的函数表 我们这里就是借用名字
先创建一个表 然后随便一条数据 然后查询测试函数即可 得到数据即添加成功
create table dual(id string);
insert into dual values(' ');
select myfunc("1902") from dual;
测试方法
select myfunc("1902");
5、删除临时的自定义函数
drop temporary function if exists myfunc ;
ps:其实不删除也可以只要退出hive就会消失 （当前session有效）

# 第二种：
1、将编译好的jar包上传到服务器
2、编辑参数文件
先跳转到hive的安装目录下创建文件
vi ./hive-init
添加下面数据
add jar /root/jar包名.jar;
create temporary function myfunc as '类的全限定名';
ps: 当前这个方式是一个批处理脚本
3、启动hive，带上初始化文件
hive -i ./hive-init
ps:若之前登录hive了 那就先退出然后在登录使用上面的命令
ps:这种方式也是相当于临时的,在直接启动hive时也是没有函数(也相当于临时函数）
                                 
# 第三种：
启动hive的时候会自动加载
1.在hive的安装目录下的bin目录下创建一个文件，文件名叫.hiverc
vi /bin/.hiverc
add jar /home/hadoop/wordcoount.jar;
create temporary function myfunc as 'moon.HiveFunction.FirstUDF';
2、启动hive
	hive
3、测试函数功能。
select myfunc("1902");

```

#### 案例2：生日转换年龄

```java
需求：输入 1980-08-10 --> 39
思路:
1.age = 当前年份-生日年
2.判断月份 ,当前鱼粉小于生日月份 age -1
3.月份相等的情况下, 判断当前日期,日期小于生日, age -1
自定义UDF函数:
1.创建一个自己的类
2.需要让这个类继承UDF --> 自定义UDF类
3.需要在类中是实现一个方法
public 返回值数据类型 evaluate(参数列表){
UDF函数的实现写到这个方法中即可
}
返回值类型: 根据实际函数需求决定返回值类型
参数列表:根据传入的参数决定个数
package com.programmer.SelfUDF;
import java.util.Calendar;
import org.apache.hadoop.hive.ql.exec.UDF;
import com.google.common.base.Strings;
/**
* UDF求年龄
* @author jkmaster
*输入 1980-08-10 --> 39
思路:
1.age = 当前年份-生日年
2.判断月份 ,当前鱼粉小于生日月份 age -1
3.月份相等的情况下, 判断当前日期,日期小于生日, age -1
*/
public class Birthday2Age extends UDF{
 public int evaluate(String birth) {
 //1.判断参数 Strings 判断是Null还是空字符串""
    if(Strings.isNullOrEmpty(birth)) {
    return -1;
    } 	
 
    //2.日期数据进行处理,拆分
String[] birthdays = birth.split("-");

    //3.因为要计算,所以需要将数据进行转换
int birthYear = Integer.parseInt(birthdays[0]);
int birthMonth = Integer.parseInt(birthdays[1]);
int birthDay = Integer.parseInt(birthdays[2]);

    //4需要用当前时间来和年月日进行计算
//Date 或 Calendar
Calendar calender = Calendar.getInstance();

    //4.1获取当年月日
int nowYear = calender.get(Calendar.YEAR);

  //Calendar 日历类比较特殊,月份是从0开始, 范围0-11 和显示时间有偏差
 //获取月份的时候需要+1 才能获取实际月份
int nowMonth = calender.get(Calendar.MONTH)+1;
    //当前月份的第几天
int nowDay = calender.get(Calendar.DAY_OF_MONTH);
	//计算年龄
int age = nowYear - birthYear;
	//判断月份和日期
    if(nowMonth < birthMonth) {
        age -= 1;
    }else if(nowMonth == birthMonth && nowDay<birthDay) {
        age -= 1;
    }
        return age;
    }
//在上传UDF函数之前,对类中所提供数据进行测试
//打包的时候需要去掉这个测方法
// public static void main(String[] args) {
// System.out.println(new Birthday2Age().evaluate("1980-09-10"));
// }
}
```

#### 案例3：日志解析

```java
//数据：
220.181.108.151 - - [31/Jan/2012:00:02:32 +0800] \"GET /home.php?mod=space&uid=158&do=album&view=me&from=space HTTP/1.1\" 200 8784 \"-\"\"Mozilla/5.0 (compatible; Baiduspider/2.0;+http://www.baidu.com/search/spider.html)\"
    
//编写UDF函数java类    
package com.programmer.SelfUDF;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import org.apache.hadoop.hive.ql.exec.UDF;
import com.google.common.base.Strings;
/**
* 日志解析：
220.181.108.151 - - [31/Jan/2012:00:02:32 +0800] \"GET /home.php?
mod=space&uid=158&do=album&view=me&from=space HTTP/1.1\" 200 8784 \"-\"
\"Mozilla/5.0 (compatible; Baiduspider/2.0;
+http://www.baidu.com/search/spider.html)\"
220.181.108.15 20120131 120232 GET /home.php?
mod=space&uid=158&do=album&view=me&from=space HTTP 200 Mozilla
*/
public class LogParse extends UDF {
public String evaluate(String log) throws ParseException {
//1.需要先判断日志字符串是否是正确
		if(Strings.isNullOrEmpty(log)) {
			return null;
			}
//匹配正则
String reg = "^([0-9.]+\\d+) - - \\[(.* \\+\\d+)\\] .+(GET|POST)(.+) (HTTP)\\S+ (\\d+) .+\\\"(\\w+).+$";
    
//31/Jan/2012:00:02:32 +0800 时间
//编译和这个正则表达式是 Pattern 是一个父类
  Pattern pattern = Pattern.compile(reg);
//匹配数据
  Matcher matcher = pattern.matcher(log);
//提供一个字符拼接对象
 StringBuffer bs = new StringBuffer();
//判断是否匹配成功
 if(matcher.find()) {
   //获取匹配成功之后的字段个数
   int count = matcher.groupCount();
   //字段数是从1开始到实际长度结束(字段数的个数)
   for(int i = 1 ; i<count;i++) {
  //匹配时间字段
   if(i == 2) {
      //20120131 120232
SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd
hhmmss");
//31/Jan/2012:00:02:32 +0800
Date d = new SimpleDateFormat("dd/MMM/yyyy:HH:mm:ssZ",Locale.ENGLISH)
.parse(matcher.group());
String s = sdf.format(d);
bs.append(s+"\t");
}else {
bs.append(matcher.group(i)+"\t");
}
}
}
return bs.toString();
}
}
```

#### 案例4：JSON数据UDF

```java
JSON信息
{"movie":"1193","rate":"5","timeStamp":"978300760","uid":"1"}
{"movie":"661","rate":"3","timeStamp":"978302109","uid":"1"}
{"movie":"914","rate":"3","timeStamp":"978301968","uid":"1"}
{"movie":"3408","rate":"4","timeStamp":"978300275","uid":"1"}
需要将数据导入到Hive数据仓库中
movie rate timeStamp uid
1193 5 978300760 1
步骤
1.创建一个单字段的表,原始JSON数据
2.创建一个自定义函数,利用函数将json解析成自己想要的格式
3.调用当前函数解析JSON传的到结果并存到表中

代码
package com.programmer.SelfUDF;
//因为JSON中数据要映射到类中的属性中
//属性名必须和JSONkey值名相同,顺序也要一致
//{"movie":"1193","rate":"5","timeStamp":"978300760","uid":"1"}
public class MovieBean {
    private String movie;
    private String rate;
    private String timeStamp;
    private String uid;
    public String getMovie() {
        return movie;
    } 
    public void setMovie(String movie) {
        this.movie = movie;
    } 
    public String getRate() {
        return rate;
    } 
    public void setRate(String rate) {
        this.rate = rate;
    } 
    public String getTimeStamp() {
        return timeStamp;
    } 
    public void setTimeStamp(String timeStamp) {
        this.timeStamp = timeStamp;
    } 
    public String getUid() {
        return uid;
    } 
     public void setUid(String uid) {
        this.uid = uid;
   } 
    public MovieBean(String movie, String rate, String 			    timeStamp, Stringuid) {
        super();
        this.movie = movie;
        this.rate = rate;
        this.timeStamp = timeStamp;
        this.uid = uid;
    } 
    public MovieBean() {
			super();
	} 
    //自定义toString方法 --> 对数据的拼接方式
    @Override
    public String toString() {
    return movie+"\t"+rate+"\t"+timeStamp+"\t"+uid;
    }
} 

package com.programmer.SelfUDF;
import java.io.IOException;
import org.codehaus.jackson.JsonParseException;
import org.codehaus.jackson.map.JsonMappingException;
import org.codehaus.jackson.map.ObjectMapper;
import com.google.common.base.Strings;
/**
* 解析JSON
* 模型对象 -->因为JSON属于键值对 可以把JSON看做是一个Map
* -->提供一个类 ,这个类是Bean 属性名和JSON串中的key名完全相同
* -->利用JSON --> kv的特点 将v值存储到对象属性中
* -->操作对象就相当于操作JSON
* -->必要前提,类中属性名和顺序,必须完全一致和JSON中key名字
* */
public class JSONParse {
    public String evaluate(String json) throws JsonParseException,JsonMappingException, IOException {
    if(Strings.isNullOrEmpty(json)) {
    return null;
    }
//ObjectMapper类是Jackson库主要类,提供一些功能个将Java对象匹配JSON结构
ObjectMapper om = new ObjectMapper();
//直接模型转换对象化
MovieBean bean = om.readValue(json, MovieBean.class);
//等价于 MovieBean mb = new MovieBean(1111,32432,32432,34543)
return bean.toString();
//1111 234324 453425 43534
} 
    public static void main(String[] args) throws JsonParseException,
JsonMappingException, IOException {
System.out.println(new JSONParse().evaluate("
{\"movie\":\"1193\",\"rate\":\"5\",\"timeStamp\":\"978300760\",\"uid\":\"1\"}"))
;
}
}    

//处理方式:hive建 表                                          
create table if not exists t_rating_json(
json string
);
load data loacl inpath '在linux系统上json存储的路径' into table t_rating_json;
将JSON自定义UDF函数打包成jar 上传到服务器上
在添加jar
add jar 'jar所在路径'
create temporary function jsonParse as 'JSON解析类的类全限定名称(包名+类名)'
                                            
//先将数据存储到一个临时表中查看效果
create table if not exists json_temp as select jsonParse(json) as json from
t_rating_json;
ps:在表汇总存储 的数据就是 "1193" "5" "978300760" "1" 数据之间是"\t"相连
                                            
//拆分结果集
select split(json,'\t')[0],split(json,'\t')[1],split(json,'\t')
[2],split(json,'\t')[3],from json_temp limit 10;
create table if not exists t_movie_rate(
        movie string,
        rate string,
        timeStamp String,
        uid string
)
 row fromat delimited fields terminated by '\t';
insert overwrite table t_movie_rate select split(json,'\t')[0],split(json,'\t')
[1],split(json,'\t')[2],split(json,'\t')[3],from json_temp
                                            
hive提供了内置JSON函数
get_json_object(json串,'$.key名字')
select get_json_object(json,'$.movie') from t_rating_json
                                            
ps:JSON文件已经被导入到表中
```

## 分区

**hive的分区实际上就是对应HDFS文件系统上独立的文件夹**,该
文件夹下是该分区所有的数据文件,其实**Hive中分区就是分目录** ,把一个大的数据集根据业务需求分隔成小的数据集,在查询的时候where子句中的表达式选择需要查询的分区,这样就可以提高效率

### 静态分区

```mysql
1.分区名严禁使用中文!!!!!! --> 使用中文会出未知错误
2.hive的分区名字区分大小写
3.hive分区本质是在表的目录下载创建一个目录
4.一张表中可以有一个或多个分区
------------------------------------------------------
# 建表格式
crate table if not exists 表名(
    列名 数据类型,
    .........
    )
partitioned by(分区名字 数据类型)
row format delimited fields terminated by '数据分隔符号';
ps:partitioned by(分区名字 数据类型),分区字段有几个就是几级分区
#加载数据格式：
load data [local] inpath '[本地|HDFS]文件存储路径' into table 表名 
partition(分区字段名字='具体的分区名称')；

#案例
#建表
create table if not exists part2(
        uid int,
        uname string,
        age int
        )
partitioned by(year string，month string)
row format delimited fields terminated by '\t';	

# 加载2017-08 数据
load data local inpath '/home/hadoop/data.txt' into table part1
partition(year='2017',month='08');
# 加载2018-06 数据
load data local inpath '/root/user-2018-06.txt' into table part2
partition(year='2018',month='06');

# 查询对应月份的数据:
select * from part2 where month='06';
# 查询对应年份中的某个月份的数据
select * from part2 where year='2017' and month='08';
# 查询整个年份中所有的数据
select * from part2 where year='2017';
# 全表扫描
select * from part2;
ps:分区还可以继续向下细分,days,但是不要过于长 基本上最大理论值(7分区)
根据业务完成即可 比较普遍的就是 时间分区 年月日 在细化小时分钟秒 (地区)
-----------------------------------------------------------------
# 显示表中分区
show partitons 表名; --> 显示表中分区数
修改分区的名字 --没有这个命令,手动暴力修改(修改HDFS文件夹的名字,后果自负)

#增加单个分区
alter table 表名 add partition(分区字段名称=分区名);
e.g :alter table part1 add partition(country='korea');

# 增加多个分区:
alter table 表名 add partition(分区字段名称=分区名) partition(分区字段名称=分区名)

# 增加分区并设置分区数据
alter table 表名 add partition(分区字段名称=分区名) location '数据路径(这个路径需要HDFS上的)'     --->路径要从hdfs上的根（/）开始
e.g : alter table part1 add partition(country='china2') location
'/user/hive/warehouse/bd1902.db/part1/data.txt';
ps: 类似于克隆因为存储是 相同文件就不会再创建新的文件夹了,但是数据是存在的

# 修改分区的HDFS存储路径
alter table 表名(分区字段名称=分区名) set location 'HDFS绝对路径'

# 删除单个分区（ 分区下有数据会被一起删除）
alter table 表名 drop partition(分区字段名称=分区名);

# 删除多个分区(删除多分区的时候要使用[,]分隔分区)
alter table 表名 drop partition(分区字段名称=分区名),partition(分区字段名称=分区名);
```

### 动态分区

```mysql
#设置严格/非严格模式： 
	set hive.exec.dynamic.partition.mode=strict/nonstrict:  严格模式/非严格模式
	ps:严格模式下，给动态分区表导入数据时，分区字段至少要有一个分区字段时静态值
	非严格模式下,导入数据时，可以不指定静态值;
#设置hive的运行模式为本地模式,在不设置之前每次MR都需要提交给Yarn
     set hive.exec.mode.local.auto = true; 默认是false值不开启,可以改为ture;
# 允许动态分区最大数量
     hive.exec.dynamic.partitions=1000
# 单个节点上mapper/reducer允许创建的最大分区数
     hive.exec.dynamic.partitions.pernode=100 	
     
  /** 
 动态分区只能使用inseret into 动态加载数据，不能用load本地加载
 */
 动态分区的值是从临时表中获取的相同字段的值来进行分区赋值

 #案例
 	1 # 创建存储数据的临时表
    create table student_temp(
        id int,
        name string,
        gender string,
        age int,
        academy string,
        year  string,
        month string,
        day string
    	)
   	 row format delimited fields terminated by ',';
    1.1 加载数据
     load data local inpath '/home/hadoop/data/student' into table student_temp; 
  2 # 创建动态分区表
  create table student_gender(
        id int,
        name string,
        gender string,
        age int,
        academy string
        )
    partitioned by (year string , month string,day string)
    row format delimited fields terminated by ',';
  3# 使用insert into加载分区
    insert into 表名 partition(分区字段名) select * from 临时存数据表的名称
    insert into student_gender partition(year,month,day) select * from  student_temp;
```



### 混合分区

```mysql
# 混合分区 : 静态加动态
其实混合分区就是 在没有修改下面这个配置的时候其实我们用的就是混合分区
set hive.exec.dynamic.partition.mode=strict;
set hive.exec.dynamic.partition.mode=nostrict;也是可以设置混合分区的的
只能使用insert into 这种语法插入数据,第二原则混合分区的第一个分区一定是一个静态分区
后续的分区可以是动态或静态分区
混合分区是 静态+动态组合所以 最少也要有一个静态和一个动态才可以成为混合分区；

1 # 建表
create table if not exists dy_st_part(
uid int,
uname string,
age int
)
partitioned by(year string,month string)
row format delimited fields terminated by '\t';

2 # 加载数据 
insert into 混合分区表名称 partition(静态分区赋值,动态分区的字段名称) select 列名 from 临时数据存储表的名字
# 查询字段要匹配(个数),静态分区字段不能出现在查询字段里面
insert into dy_st_part partition(year='2017',month) select uid,uname,age,month from part2 where year='2017';

```

### 总结

```mysql
总结:
1 静态分区是通过load进行赋值，动态分区和混合分区是使用insert into进行赋值的

2 动态分区需要修改下面属性（没设置下列属性之前）
set hive.exec.dynamic.partition.mode=strict; 必须是一个静态和一个动态
set hive.exec.dynamic.partition.mode=nostrict; 不需要一定有静态分区（全局动态）

3 混合分区 = 静态+动态分区,第一个分区需要是静态分区,后续可以是静态或动态,
  混合分区最低要求是一个静态+一个动态,赋值的时候,静态分区字段不能出现在查询语句的字段     中,动态分区的字段根据查询语句中相同字段进行赋值

 4 静态分区：通过load方式加载数据,然后在加载数据的同时指定分区
 
   动态分区 ：需要设置权限,开启全模式动态分区,通过 insert into 动态加载数据
   set hive.exec.dynamic.partition.mode=nostrict
   
   混合分区：静态分区+动态分区,第一个分区值需要是静态的,第二分区是动态(或静态)
            最低要求：一个静态一个动态，数据通过insert into形式加载
            加载数据过程中 静态分区指定分区，动态分区不指定
 ----------------------------------------------------------------------------
#设置严格/非严格模式： 
	set hive.exec.dynamic.partition.mode=strict/nonstrict:  严格模式/非严格模式
	ps:严格模式下，给动态分区表导入数据时，分区字段至少要有一个分区字段时静态值
	非严格模式下,导入数据时，可以不指定静态值;
	
#设置hive的运行模式为本地模式,在不设置之前每次MR都需要提交给Yarn
     set hive.exec.mode.local.auto = true; 默认是false值不开启,可以改为ture;
     
# 允许动态分区最大数量
     hive.exec.dynamic.partitions=1000
     
# 单个节点上mapper/reducer允许创建的最大分区数
     hive.exec.dynamic.partitions.pernode=100 
     
#设置强制分桶属性(默认不开启，设置之后所有建表都是分桶的)
set hive.enforce.bucketing=false/true

#设置reduce的个数为n 建立分桶表必须设置reduce个数
set mapreduce.job.reduces=n
```



## 分桶

```mysql
# 格式：
create table if not exists 表名(
列名 列的数据类型
.....
)
clustered by(分桶字段) [sorted by(桶内的数据什么列进行排序 ASC|DESC)] into 数值(桶的个数) buckets
ps: 排序不是一定要加的,需要桶内数据有序,可以添加

#设置强制分桶属性(默认不开启，设置之后所有建表都是分桶的)
set hive.enforce.bucketing=false/true
#设置reduce的个数为n 建立分桶表必须设置reduce个数
set mapreduce.job.reduces=n 

# 分桶的原理：
分桶的列的hash值对桶的个数取模，相同的为一桶，

#普通表分桶查询案例   
# 创建一张临时数据表  ----普通建表但是查询时候分桶查询 
create table t_stu(
son int,
sname string,
sex string,
sage int,
sdept string
)
row format delimited fields terminated by ',';
#加载数据
load data local inpath '/root/students.txt' into table t_stu;

 # 分桶查询 需要对reduce个数设置 
set mapreduce.job.reduces=n #设置reduce的个数为n

mapreduce.job.reduces=-1    #代表的是默认值 即开启一个reduce
select 列名 from 表名 cluster by(需要分桶的列)
select * from t_stu cluster by(son);

# cluster by-->查询语句 桶内的数据记录是有序的，但是是升序
select * from t_stu cluster by(son); 等于
select * from t_stu distribute by(son) sort by(son ASC)

---------------------------------------------------------------------------------
#建立分桶表案例
# 创建一个分桶表 桶按照sno分 排序是 sage desc 分4桶 将数据加载进来
create table buc1(
    son int,
    sname string,
    sex string,
    sage int,
    sdept string
)
clustered by(son) sorted by(sage desc) into 4 buckets
row format delimited fields terminated by ',';

# 加载数据（动态加载，使用load加载不能体现分桶效果）
 加载数据的时候需要使用临时表,这个临时表已经创建了 t_stu
insert into table buc1 select * from t_stu cluster by(son);
#如果想要根据son降序，只能使用下面这种方法
# cluster by（） 只能升序
insert into table buc3 select * from t_stu distribute by(son) sort by(sage
desc);	
--------------------------------
	#分桶抽样查询
    # 查询全部桶中数据
    select * from 表名 --> 桶中的所有数据
    
    # 分桶抽样查询语句  （x是第几桶 y是总桶数）
    select * from 表名 tablesample(bucket x out of y on 分桶的列);
       #查询第一桶数据
       select * from buc1 tablesample(bucket 1 out of 4 on son);
       #查询第一桶和第三桶数据
  	   select * from buc1 tablesample(bucket 1 out of 2 on son);
    y的值是总桶数,所以可以对y值进行改变,这个改变必须是y的因子或倍数
    x表示的是桶数 y值是当前的总桶数的因子值(整数)
    计算公式 x+(s/y)*(n-1)
    x 代表是第几桶 s总桶数 y现在查询桶数(总桶数的因子) n是次数(从1开始逐渐递增)
    压缩抽样查询
    1 2 3 4
    1 2 1 2
    第一次获取值: x+(s/y)*(n-1) = 1+(4/2)*(1-1)=1+2*0 = 1
    第二次获取值: x+(s/y)*(n-1) = 1+(4/2)*(2-1)=1+2*1 = 3
    第三次获取值: x+(s/y)*(n-1) = 1+(4/2)*(3-1)=1+2*2 = 5
    5值已经大于总桶个数所以停止获取
    select * from buc3 tablesample(bucket 1 out of 8 on son)
    y的值是总桶数,所以可以对y值进行改变,这个改变必须是y的倍数
    原有存储在桶中数据会进行拉伸
    拉伸抽样查询
    y的值等于8 是4的倍数 相当于扩大了2倍
    1 2 3 4
    5 6 7 8
    原来桶中数据是 son%4 = 余数值 决定了在哪个桶中
    现在拉伸之后实这样计算 son%8 = 余数值
    将原有存储在1桶中值会被拆分到 1桶和5桶中
    抽取的时候(4/8) =1/2 相当于桶中数据源原有桶中数据的半
    
    #查询son为奇数数据
    select * from buc1 tablesample(bucket 2 out of 2 on son);
    #查询son中偶数中名字带有庆
    select * from buc1 tablesample(bucket 1 out of 2 on son) where sname like '%庆%';
    #通过抽样查询 limit 限制
    select * from buc1 limit 3;
    select * from buc1 tablesample(3 rows);
    #使用百分比的形式来查询
    select * from buc1 tablesample(13 percent);
    #根据数据进查询的,无论如何查询 一行数据就是行
    select * from buc1 tablesample(68B); -->数据类型可以是 K KB M G T P
    #要求随机抽取3行数据：
    select * from t_stu order by rand() limit 3;
```





