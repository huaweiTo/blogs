# spark编程

**教材** Spark编程基础 scala 版 林子雨等 人民邮电

## Spark SQL

1. 代码方面  
   1. 写代码时，出错，首先 看看变量类型，该变量没有这个方法。IDEA中自动填充变量类型快捷键，alt + enter  

    2. show() 方法默认是只显示20个，可以指定行数，sum(num,boolean),num指的是显示几行，boolean指的是，每行过长时是否截断，false是不截断，默认为true  

    3. 自定义在从RDD隐式转换为DF编程中，case class要放在方法外，比如我的Object类里就写了一个main方法，我在main方法里写所有代码，那么case class 要放在main方法外面  

    4. 隐式转换 `import spark.implicits._ `这句话是下载代码中的SparkSession对象下面的，比如  
    `val s= SparkSession.builder()`  
    那么导包就得写为`import s.implicits._`

    5. sortBy()使用没效果，看是不是设置的local[n],这样会分开n个区操作，每个区排序会导致排序不对。设置为local或者local[1]就可以了

    6. 创建表头，  
    ` val fields = Array(StructField("name",StringType,true), StructField("age",IntegerType,true))`  
    别把StringType的包导错  

2. 使用Spark SQL读写数据库的时候  
   1. 提示SSL=false  则需要修改其中一语句如下  
   `.option("url","jdbc:mysql://localhost:3306/spark?useSSL=false")`  

   2. 报错 Access Denied   
   先看你的mysql数据库里的用户名和密码对应问题
   见[博客1](https://blog.csdn.net/qq1319713925/article/details/84979569)  
   操作为在mysql中运行  
    `use mysql;`  
    `select host,user,authentication_string from user;查看是否类似下面的情况`  
    有多个root对应，可以删除,如  
    `delete from user where host='127.0.0.1'`  

3. 连接Hive读写数据  
   1. hive使用过程中特别慢，报错，  
   需要把hive-site.xml配置文件中的mysql用户名和密码设置成自己的，默认全是hive  

   2. spark操作失败，报错  
   `org.apache.spark.sql.AnalysisException: Table or view not found: `sparktest`.`student`; line 1 pos 14;`  
   看是不是没把hive的配置文件hive-site.xml拷贝到spark下，我电脑上路径为`/usr/local/spark/conf`
