使用系统自带的日志切割工具logrotate切割

vi /etc/logrotate.d/tomcat  
/usr/local/apache-tomcat/logs/catalina.out {  
copytruncate  
daily  
rotate 5  
missingok  
compress  
size 16M  
}

配置简单说明：  
/usr/local/apache-tomcat/logs/catalina.out{ \#要切割的文件  
daily \# 每天进行catalina.out文件的切割  
rotate 5 \# 保留5个文件  
missingok \# 文件丢失了，继续切割而不报错  
compress \# 使用压缩的方式  
size 16M \# 当catalina.out文件大于16MB时，就切割

工作原理：  
每天晚上crond守护进程会运行在/etc/cron.daily目录中的任务列表；  
与logrotate相关的脚本也在/etc/cron.daily目录中。运行的方式为"/usr/bin/logrotate /etc/logrotate.conf"；  
/etc/logrotate.conf文件include了/etc/logrotate.d/目录下的所有文件。还包括我们上面刚创建的tomcat文件；  
/etc/logrotate.d/tomcat文件会触发/usr/local/apache-tomcat/logs/catalina.out文件的轮转。  


