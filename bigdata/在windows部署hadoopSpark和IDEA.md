# 在windows本地编写spark程序的所需环境

为了能在windows使用IDEA编写spark代码，本地跑成功后提交到虚拟机的Ubuntu系统（已经安装hadoop等所有所需的软件）中运行，尝试了以下步骤：  

1. Ubuntu中对于maven下载的jar包，可以找到IDEA设置中maven的库文件夹——repository目录，通过FTP客户端直接拷贝到windows下，如D:\maven\repository。随后将windows端IDEA中的maven的repository设置指向D:\maven\repository。这样可以省去重新下载jar包的步骤。

2. maven及导包后，代码可以正常编写，但是运行，会提示  
`java.io.IOException: Could not locate executable null\bin\winutils.exe in the Hadoop binaries.`  
这是因为本地没有安装hadoop的原因。  
[博客1](https://www.cnblogs.com/java-spring/p/11744195.html)中给出了详细步骤，按照运行就行。  
[博客2](https://blog.csdn.net/qq_35535690/article/details/81976032)给出了具体设置步骤，按照操作即可。  
同时所需的配置文件内容可以参考[博客3](https://www.cnblogs.com/timlong/p/9991662.html)。  

3. 以上完成后，再次运行IDEA，报错：  
`Exception in thread "main" java.lang.IllegalArgumentException: Unsupported class file major version 56`  
通过查阅[博客4](https://www.cnblogs.com/liujStudy/p/7217480.html)，可知这是因为spark和Java版本支持问题。通过查看IDEA编译的class文件可知，该class文件为version 52，即java 8，而我window装的时java12，即version 56。需要：  
卸载Windows已经安装的java，并重装Java 8。  

现在IDEA终于可以在windows跑成功了。



