# hive使用语句脚本
用的是databases,是虚拟机hive中的sparktest

## 动态分区示例

create table if not exists salary(name string,age int,salary int) row format delimited fields terminated by ',';

load data local inpath '/home/hadoop/桌面/Spark编程代码练习/spark花航sql题目/spark_data.txt' into table salary;

create table if not exists salary_dynamic_partition(name string,salary int) partitioned by (age int) row format delimited fields terminated by ',';



set hive.exec.dynamic.partition=true;    //开启
set hive.exec.dynamic.partition.mode=nonstrict;      //设置分区模式为非严格模式

insert overwrite table salary_dynamic_partition partition(age)
select name,salary,age from salary;

不能用
load data local inpath '/home/hadoop/桌面/Spark编程代码练习/spark花航sql题目/spark_data.txt' overwrite table salary_dynamic_partition partition(age_1=age);

## 分桶表示例
create table salary_buck(name string,age int, salary int) 
clustered by(age) 
into 32 buckets
row format delimited fields terminated by ',';

set hive.enforce.bucketing=true;

insert into table salary_buck 
select * from salary;

抽样查询
select * from salary_buck tablesample(bucket 1 out of 2 on age);

## 列转行
  1 桃姐    剧情/家庭
  2 好友请求        惊悚/恐怖
  3 网络谜踪        剧情/悬疑/惊悚/犯罪
  4 卧虎藏龙        剧情/爱情/武侠
  5 剑雨     动作/爱情/武侠/古装
  6 蜀山    剧情/动作/奇幻/武侠
  7 富贵逼人        喜剧/家庭/奇幻
  8 夏洛特烦恼      喜剧/爱情
  9 普罗旺斯的夏天  剧情/喜剧/家庭
create table movie_info(
movie string,
category array<string>)
row format delimited fields terminated by '\t'
collection items terminated by '/';

load data local inpath '/home/hadoop/桌面/Spark编程代码练习/hive命令练习/file.txt' into table movie_info;

按需求查询数据
select movie,category_name
from 
movie_info lateral view explode(category) table_tmp as category_name;

## 窗口函数
create table business(
name string,
orderdate
string,cost int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

load data local inpath "/home/hadoop/桌面/Spark编程代码练习/hive命令练习/over相关.txt" into table business;

select name,count(*) over()
from business
where substring(orderdate,1,7) = '2017-04'
group by name;

select name,orderdate,cost,sum(cost) over(partition by month(orderdate)) from
business;
select name,orderdate,cost,sum(cost) over(partition by day(orderdate)) from business;

select name,orderdate,cost,
sum(cost) over() as sample1,--所有行相加
sum(cost) over(partition by name) as sample2,--按name 分组，组内数据相加
sum(cost) over(partition by name order by orderdate) as sample3,--按name 分组，组内数
sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING
and current row ) as sample4 ,--和sample3 一样,由起点到当前行的聚合
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING and current
row) as sample5, --当前行和前面一行做聚合
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING AND 1
FOLLOWING ) as sample6,--当前行和前边一行及后面一行
sum(cost) over(partition by name order by orderdate rows between current row and
UNBOUNDED FOLLOWING ) as sample7 --当前行及后面所有行
from business;
