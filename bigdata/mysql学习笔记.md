# mysql学习笔记

1. 用到同一张表的两个参数时可以把同一张表用两次吗，名为a,b，进行join，可以不用on 直接where ，是自连接，即连接的是自己的一个镜像。  

2. SQL语句的执行顺序，从[博客1](https://www.php.cn/sql/421993.html)和[博客2](https://baijiahao.baidu.com/s?id=1655862131794837762&wfr=spider&for=pc)可知，顺序为  
```sql
(8) SELECT     (9)DISTINCT<select_list>
(1) FROM <left_table>
(3) <join_type> JOIN <right_table>
(2) ON <join_condition>
(4) WHERE <where_condition>
(5) GROUP BY <group_by_list>
(6) WITH {CUBE|ROLLUP}
(7) HAVING <having_condition>
(10) ORDER BY <order_by_list>
(11) LIMIT <limit_number>
```
也就是`from-on-join-where-groupby-with-having-select-distinct-orderby-limit`

3. 如果多张表，一定记得把多表中存在的字段加上表明，即表名.字段名  

4.   replace 用法  
将id=5以及emp_no=10001的行数据替换成id=5以及emp_no=10005,其他数据保持不变，使用replace实现。
update titles_test set emp_no=replace(emp_no,10001,10005) where id=5
-- 另外
 将address字段里的 “区” 替换为 “呕” 显示，如下
select *,replace(address,'区','呕') AS rep  
from test_tb  
将address字段里的 “东” 替换为 “西” ，如下  
update test_tb set address=replace(address,'东','西') where id=2  
将id=6的name字段值改为wokou  
replace into test_tb VALUES(6,'wokou','新九州岛','日本')  
向表中“替换插入”一条数据，如果原表中没有id=6这条数据就作为新数据插入(相当于insert into作用)；如果原表中有id=6这条数据就做替换(相当于update作用)。对于没有指定的字段以默认值插入。  

5. concat()函数
功能：将多个字符串连接成一个字符串。  
语法：concat(str1, str2,...)  
返回结果为连接参数产生的字符串，如果有任何一个参数为null，则返回值为null。

6. concat_ws()函数
功能：和concat()一样，将多个字符串连接成一个字符串，但是可以一次性指定分隔符～（concat_ws就是concat with separator）  
语法：concat_ws(separator, str1, str2, ...)  
说明：第一个参数指定分隔符。需要注意的是分隔符不能为null，如果为null，则返回结果为null。

7. group_concat()函数 见[博客3](https://baijiahao.baidu.com/s?id=1595349117525189591&wfr=spider&for=pc)
功能：将group by产生的同一个分组中的值连接起来，返回一个字符串结果。  
语法：group_concat( [distinct] 要连接的字段 [order by 排序字段 asc/desc ] [separator '分隔符'] )  
说明：通过使用distinct可以排除重复值；如果希望对结果中的值进行排序，可以使用order by子句；separator是一个字符串值，缺省为一个逗号。

8. substr(X,Y,Z) 或 substr(X,Y) 函数的使用。其中X是要截取的字符串。Y是字符串的起始位置（注意第一个字符的位置为1，而不为0），取值范围是±(1~length(X))，当Y等于length(X)时，则截取最后一个字符；当Y等于负整数-n时，则从倒数第n个字符处截取。Z是要截取字符串的长度，取值范围是正整数，若Z省略，则从Y处一直截取到字符串末尾；若Z大于剩下的字符串长度，也是截取到字符串末尾为止。  
SELECT first_name FROM employees ORDER BY substr(first_name,length(first_name)-1)
也可  
SELECT first_name FROM employees ORDER BY substr(first_name,-2) 

9. 窗口函数  
   1. MySQL和sqlite中，rank dense_rank 允许并列排名，rank并列的数值占用排名，dense_rank不占用排名,row_number 不允许并列  
   ![rank窗口函数比较](https://pic4.zhimg.com/v2-c08d436f60b47663ac4c3b980cba4e1e_r.jpg)  

   2. SUM OVER PARTITION BY ORDER BY(分组累计计算方法)  
   其他窗口函数组合使用见[博客4](https://blog.csdn.net/lidengm/article/details/83543081)  

10. case 语句  
    奖金金额bonus。 bonus类型btype为1其奖金为薪水salary的10%，btype为2其奖金为薪水的20%，其他类型均为薪水的30%。 
    ```sql
    select (
        case btype
            when 1 then salary*0.1
            when 2 then salary*0.2
            else salary*0.3
        end ) as bonus
    ```

## 其他
查看MySQL中hive用户权限  
  `use mysql;`  
    `select host,user,authentication_string from user;查看是否类似下面的情况`  

发现权限一点也没有，重新授权，但是报错，说root用户没有权限，  
`select * from mysql.user where User='root' and Host='%'\G;`   
确实能看到Grant_priv: N  
使用命令使root用户拥有给其他用户赋值的能力，
` update mysql.user set Grant_priv='Y' where User='root' and Host='%';`  
刷新`flush privileges;`重新登录  
成功  

创建用户  
`CREATE USER 'winUser'@'192.168.%' IDENTIFIED BY 'hadoop';`  

`GRANT ALL ON *.* TO  'winUser'@'192.168.%';`  
`flush privileges;（刷新权限）`  