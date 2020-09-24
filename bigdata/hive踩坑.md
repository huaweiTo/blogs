# HIVE

## 踩坑

本来hive运行正常，但是由于运行试验spark sql期间把mysql中的所有root相关用户都删除了，只剩一个，且把hive-site.xml配置文件中的连接MySQL用户名和密码都改成了root，所以这次在使用hive时，运行show databases等命令报错
`MetaException(message:Hive Schema version 2.3.0 does not match metastore's schema version 1.2.0 Metastore is no`。  
通过[博客1](https://blog.csdn.net/qq_39579408/article/details/86526757)和[博客2](https://blog.csdn.net/struggling_rong/article/details/82598277)可知，
原因为： spark应用创建表时，指定的schema版本为1.2.0，而hive的schema版本为2.3.0，版本不兼容导致。  
解决为： 在hvie-site.xml中将hive.metastore.schema.verification参数设置为false
```
<property>
    <name>hive.metastore.schema.verification</name>
    <value>false</value>
</property>
```
成功

## 理论知识
1. 分区作用。  
由[博客2](https://www.jianshu.com/p/64bfd488d2f0)如果没有分区的存在，那么每次查询Hive将会进行全表扫描。对于小数据量的表来说，全表扫描并不会慢到无法忍受，但是对于大数据量来讲，比如几年的数据，每次查询都要扫描几年的所有数据，除了浪费时间之外，还浪费集群资源。为了改进这一问题，分区的价值就体现出来了。  
   1. 静态分区  
   Hive的静态分区，实际上就是手动指定分区的值为静态值，这种对于小批量的分区插入比较友好，语句中partition(year=“2020”, month=“04”, day=“2020-04-10”, hour=“22”) 的年月日小时手动指定了具体的值，这样的分区就叫静态分区了。
   2. 动态分区  
   Hive的动态分区，其实就是把静态分区中的分区值设置为动态的值，就可以了
   ```sql
    insert overwrite table demo_dynamic_partition 
    partition(year=year, month=month, 
    day=day, hour=hour) 
    select user_id, user_name, 
    trade_year as year ,
    trade_month as month,
    trade_day as day,
    trade_hour as hour  
    from user_demo 
   ```
   语句中partition(year=year, month=month, day=day, hour=hour)会根据具体值的变化而变化，无需手动指定，这对于大批量的分区插入是一个很方便的用法，但需要根据业务需求衡量分区数量是否合理的问题。毕竟分区会占用IO资源，数量越多，IO资源消耗越大，查询时间和性能都是有所损耗的。
   2. 结合使用  
   Hive静态分区和动态分区结合使用
   Hive的动态分区，其实就是把静态分区中的分区值设置为动态的值，就可以了
   ```sql
    partition(year="2020", month="04", 
    day=day, hour=hour) 
    select user_id, user_name, 
    trade_year as year ,
    trade_month as month,
    trade_day as day,
    trade_hour as hour  
    from user_demo 
    where trade_year="2020" 
    and trade_month="04" 
   ```

不难看出，Hive分区，主要是以缩小数据查询范围，提高查询速度和性能的。

2. 分桶表创建后，直接load data不会有分桶的效果，这样和不分桶一样，在HDFS上只有一个文件。于是需要创建分桶表时，设置参数set hive.enforce.bucketing=true，并且数据通过子查询的方式导入，这样就会在hdfs上分成几个文件。  
[博客3](http://www.voidcn.com/article/p-xjvzjzns-bwe.html)中解释抽样查询  
SELECT avg(viewTime)
FROM page_view TABLESAMPLE(BUCKET 1 OUT OF 32);  

 >当您使用clustered子句创建表，并将其存储到32个桶中时(作为示例)，hive使用确定性散列函数将您的数据存储到32个桶中。然后当你使用TABLESAMPLE(BUCKET x OUT OF y)时,hive将你的桶分成y个桶组（每组有y个桶），然后选择每个组的第x个桶，也就是整体的第x + y个桶。例如：如果您使用 TABLESAMPLE ( BUCKET 6 OUT OF 8 ) ，蜂巢会将您的32个桶分成8个桶组，产生4组8个桶，然后挑选每组的第6个桶，因此挑选桶6,14,22 ,30。如果您使用TABLESAMPLE ( BUCKET 23 OUT of 32 ) ，hive会将您的32个桶分成32个组，只产生1组32个桶，然后选择第23个桶作为结果。如果你使用TABLESAMPLE ( BUCKET 3 out of 64 ) ，hive会将你的32个桶分成64个桶组，产生一组64个“半桶”，然后选择对应于第3个满桶的半桶-桶。

3. HiveQL与SQL区别  
    从[博客4](https://www.cnblogs.com/bjlhx/p/6946242.html)可知
   1. Hive不支持等值连接   
    SQL中对两表内联可以写成：
    select * from dual a,dual b where a.key = b.key;
    Hive中应为
    select * from dual a join dual b on a.key = b.key; 
    而不是传统的格式：
    SELECT t1.a1 as c1, t2.b1 as c2FROM t1, t2
    WHERE t1.a2 = t2.b2  
    2. IS [NOT] NULL
     SQL中null代表空值, 值得警惕的是, 在HiveQL中String类型的字段若是空(empty)字符串, 即长度为0, 那么对它进行IS NULL的判断结果是False.  
     3. hive支持嵌入mapreduce程序，来处理复杂的逻辑  
     4. hive支持将转换后的数据直接写入不同的表，还能写入分区、hdfs和本地目录
      这样能免除多次扫描输入表的开销。

4. SQL 必学函数  
   1. concat(str1,SEP,str2,SEP,str3,……) 和 concat_ws(SEP,str1,str2,str3, ……)   字符串连接函数,需要是 string型字段
   2. from_unixtime(unix_timestamp(),'yyyy-MM-dd HH:mm:ss') 时间转换函数[博客5](https://blog.csdn.net/mingming20547/java/article/details/94596351 )
            1、unix_timestamp() 得到当前时间戳 
            如果参数date满足yyyy-MM-dd HH:mm:ss形式，则可以直接unix_timestamp(string date) 得到参数对应的时间戳 
            如果参数date不满足yyyy-MM-dd HH:mm:ss形式，则我们需要指定date的形式，在进行转换
        >select unix_timestamp('2009-03-20')   --1237507200
        >select unix_timestamp('2009-03-20 00:00:00', 'yyyy-MM-dd HH:mm:ss') --1237507200
        >select unix_timestamp('2009-03-20 00:00:01', 'yyyy-MM-dd HH:mm:ss') --1237507201
        >select unix_timestamp('2009/03/20 00:00:01', 'yyyy-MM-dd HH:mm:ss') --NULL  
            2.语法：from_unixtime(t1,’yyyy-MM-dd HH:mm:ss’) 
            其中t1是10位的时间戳值，即1970-1-1至今的秒，而13位的所谓毫秒的是不可以的。 
            对于13位时间戳，需要截取，然后转换成bigint类型，因为from_unixtime类第一个参数只接受bigint类型。例如： 
            select from_unixtime(1237507201,'yyyy-MM-dd HH:mm:ss') -- 2009-03-20 00:00:01
        >select from_unixtime(1237507200,'yyyy-MM-dd HH:mm:ss') -- 2009-03-20 00:00:00
        >常用的插入时间为当前系统时间 转换成距离1970的时间戳，再转换成当前时间
        >select FROM_UNIXTIME(UNIX_TIMESTAMP() ,'yyyy-MM-dd HH:mm:ss') AS W_INSERT_DT
   3.  regexp_replace(string A, string B, string C) 字符串替换函数
            按照Java正则表达式PATTERN将字符串INTIAL_STRING中符合条件的部分成REPLACEMENT所指定的字符串，如里REPLACEMENT这空的话，抽符合正则的部分将被去掉  如：regexp_replace("foobar", "oo|ar", "") = 'fb.' 注意些预定义字符的使用，如第二个参数如果使用'\s'将被匹配到s,'\\s'才是匹配空格
   4. to_date(string timestamp) 将时间戳转换成日期型字符串
   5. date_add(string startdate, int days) 日期加减
    ```shell
        hive (default)> select date_add('2020-07-20',7);
        OK
        _c0
        2020-07-27
        Time taken: 6.524 seconds, Fetched: 1 row(s)
        hive (default)> select date_sub('2020-07-20',7);
        OK
        _c0
        2020-07-13
        Time taken: 0.342 seconds, Fetched: 1 row(s)
    ```
   6. date_format(date/timestamp/string ts, string fmt) 日期格式
        返回日期时间字段中的日期部分
        ```shell
            hive (default)> select date_format('2020-07-20','yyyy/MM/dd');
            OK
            _c0
            2020/07/20
            Time taken: 0.11 seconds, Fetched: 1 row(s)
            hive (default)> select date_format('2020-07-20','yyyyMMdd');
            OK
            _c0
            20200720
        ```
   7. last_day(string date) 返回 当前时间的月末日期
            ```shell
                    hive (default)> select  last_day('2020-07-22 10:03:01');
                    OK
                    _c0
                    2020-07-31
            ```
   8. if(boolean testCondition, T valueTrue, T valueFalseOrNull) ，根据条件返回不同的值
            说明:  当条件testCondition为TRUE时，返回valueTrue；否则返回valueFalseOrNull`
            hive> select if(a=a,’bbbb’,111) fromlxw_dual;
             bbbb
            hive> select if(1<2,100,200) fromlxw_dual;
             200
   9. nvl(T value, T default_value) 如果T is null ，返回默认值
            ```shell
                         hive (default)> select nvl(1,23);
                         OK
                         _c0
                         1
                         Time taken: 0.073 seconds, Fetched: 1 row(s)
                         hive (default)> select nvl(null,23);
                         OK
                         _c0
                         23
            ```
   10.  length(string A) 返回字符串A的长度	
   11. split(str, regex) ,安装规则截取字符串,返回数组
   ```shell
                hive (default)> select split('帅哥|$美女','\\|\\$')[0];
                OK
                _c0
                帅哥
                Time taken: 0.097 seconds, Fetched: 1 row(s)
                hive (default)> select split('ab-cd','-');
                OK
                _c0
                ["ab","cd"]
    ```
   12. 常用窗口函数
    row_number():从1开始，按照顺序，生成分组内记录的序列,row_number()的值不会存在重复,当排序的值相同时,按照表中记录的顺序进行排列;通常用于获取分组内排序第一的记录;获取一个session中的第一条refer等。
    rank()：生成数据项在分组中的排名，排名相等会在名次中留下空位。
    dense_rank():生成数据项在分组中的排名，排名相等会在名次中不会留下空位

