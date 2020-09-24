# hiveʹ�����ű�
�õ���databases,�������hive�е�sparktest

## ��̬����ʾ��

create table if not exists salary(name string,age int,salary int) row format delimited fields terminated by ',';

load data local inpath '/home/hadoop/����/Spark��̴�����ϰ/spark����sql��Ŀ/spark_data.txt' into table salary;

create table if not exists salary_dynamic_partition(name string,salary int) partitioned by (age int) row format delimited fields terminated by ',';



set hive.exec.dynamic.partition=true;    //����
set hive.exec.dynamic.partition.mode=nonstrict;      //���÷���ģʽΪ���ϸ�ģʽ

insert overwrite table salary_dynamic_partition partition(age)
select name,salary,age from salary;

������
load data local inpath '/home/hadoop/����/Spark��̴�����ϰ/spark����sql��Ŀ/spark_data.txt' overwrite table salary_dynamic_partition partition(age_1=age);

## ��Ͱ��ʾ��
create table salary_buck(name string,age int, salary int) 
clustered by(age) 
into 32 buckets
row format delimited fields terminated by ',';

set hive.enforce.bucketing=true;

insert into table salary_buck 
select * from salary;

������ѯ
select * from salary_buck tablesample(bucket 1 out of 2 on age);

## ��ת��
  1 �ҽ�    ����/��ͥ
  2 ��������        ���/�ֲ�
  3 ��������        ����/����/���/����
  4 �Ի�����        ����/����/����
  5 ����     ����/����/����/��װ
  6 ��ɽ    ����/����/���/����
  7 �������        ϲ��/��ͥ/���
  8 �����ط���      ϲ��/����
  9 ������˹������  ����/ϲ��/��ͥ
create table movie_info(
movie string,
category array<string>)
row format delimited fields terminated by '\t'
collection items terminated by '/';

load data local inpath '/home/hadoop/����/Spark��̴�����ϰ/hive������ϰ/file.txt' into table movie_info;

�������ѯ����
select movie,category_name
from 
movie_info lateral view explode(category) table_tmp as category_name;

## ���ں���
create table business(
name string,
orderdate
string,cost int
) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

load data local inpath "/home/hadoop/����/Spark��̴�����ϰ/hive������ϰ/over���.txt" into table business;

select name,count(*) over()
from business
where substring(orderdate,1,7) = '2017-04'
group by name;

select name,orderdate,cost,sum(cost) over(partition by month(orderdate)) from
business;
select name,orderdate,cost,sum(cost) over(partition by day(orderdate)) from business;

select name,orderdate,cost,
sum(cost) over() as sample1,--���������
sum(cost) over(partition by name) as sample2,--��name ���飬�����������
sum(cost) over(partition by name order by orderdate) as sample3,--��name ���飬������
sum(cost) over(partition by name order by orderdate rows between UNBOUNDED PRECEDING
and current row ) as sample4 ,--��sample3 һ��,����㵽��ǰ�еľۺ�
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING and current
row) as sample5, --��ǰ�к�ǰ��һ�����ۺ�
sum(cost) over(partition by name order by orderdate rows between 1 PRECEDING AND 1
FOLLOWING ) as sample6,--��ǰ�к�ǰ��һ�м�����һ��
sum(cost) over(partition by name order by orderdate rows between current row and
UNBOUNDED FOLLOWING ) as sample7 --��ǰ�м�����������
from business;
