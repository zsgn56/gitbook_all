jmetter 400并发压测数据，节点数量2个，soms容器数量2个，压测soms相关流程![](/assets/2018.10.301.png)![](/assets/201810302.png)问题现象：soms连接sgbs日志超时，sgbs日志本身无异常

原因：   soms上所的节点线程数达到默认限制32768

解决办法：修改最大线程数



2

jmetter 400并发压测数据，节点数量2个，soms容器数量2个，压测soms相关流程![](/assets/201811022.png)

![](/assets/201811201.png)问题现象：soms连接其它项目日志超时，smos项目进程异常退出

原因：   数据库占cpu

解决办法：修改最大acs的数据库查询语句
