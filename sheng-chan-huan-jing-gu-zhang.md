#### 1、多个dubbo注册服务提示连接超时，ssps、crm、compass服务注册的服务少了一个。

![](/assets/dubbo-error.png)

![](/assets/dubbo-abs.png)

处理过程：重启abs

原因未找到，soms、tbs项目出现过OutOfMemoryError，怀疑是节点主机资源出现异常，导致注册的服务不正常。

4.5后续发现同样情况，某台节点阿里云系统容器占资源，重启该节点，另外有连接lts报错，重启了lts服务。

4.6 后续发现服务器无法连接，提示fork retry Resource temporarily unavailable，重启后恢复

**4.12 ps -eLf \| wc -l 观察查看是由于abs线程数没有释放，达到内核32768的线程限制，是程序有配置连接APM监控，但实际上没有启用该工作，导致连接没有释放。**

附：

查看最大进程数 sysctl kernel.pid\_max

ps -eLf \| wc -l查看进程数

修改最大进程数后系统恢复

echo 1000000 &gt; /proc/sys/kernel/pid\_max

永久生效

echo "kernel.pid\_max=1000000 " &gt;&gt; /etc/sysctl.conf

sysctl -p

#### 2、多个dubbo注册服务提示连接超时，activity\sso 等网站不能访问，

检查wxs服务报错：2018-10-22 11:04:29,951 \[DubboServerHandler-172.18.136.13:28201-thread-104\] ERROR c.a.dubbo.rpc.filter.ExceptionFilter -  \[DUBBO\] Got unchecked and undeclared exception which called by 172.18.29.15. service: com.hk.wxs.api.IServiceCustomServiceService, method: sendMessage, exception: com.hk.wxs.interfaces.exception.RemoteException: weixin return error:45015, msg:response out of time limit or subscription is canceled hint: \[GUaS.a0259szb1\], dubbo version: 2.8.4, current host: 172.18.136.13

com.hk.wxs.interfaces.exception.RemoteException: weixin return error:45015, msg:response out of time limit or subscription is canceled hint: \[GUaS.a0259szb1\]

检查ubs报错：

![](/assets/2018102301.png)

\#\#\# Error updating database.  Cause: java.sql.SQLException: Lock wait timeout exceeded; try restarting transaction

\#\#\# The error may involve com.hk.ubs.mapper.UserOauthMapper.insertSelective-Inline

**总结：**未确认原因，怀疑是数据吹插入失败，或者微信提供方服务异常

#### 3、生产环境服务器被入侵挖矿病毒

参考：[https://my.oschina.net/u/3473218/blog/3013593](https://my.oschina.net/u/3473218/blog/3013593)

[http://www.nxadmin.com/penetration/1590.html](http://www.nxadmin.com/penetration/1590.html)

重点：通过docker获取busybox，清理脚本

```
#!/bin/bash
service crond stop

busybox rm -f /etc/ld.so.preload
busybox rm -f /usr/local/lib/libcset.so
chattr -i /etc/ld.so.preload
busybox rm -f /etc/ld.so.preload
busybox rm -f /usr/local/lib/libcset.so

# 清理异常进程
busybox ps -ef | busybox grep -v grep | busybox egrep 'ksoftirqds' | busybox awk '{print $1}' | busybox xargs kill -9
busybox ps -ef | busybox grep -v grep | busybox egrep 'kthrotlds' | busybox awk '{print $1}' | busybox xargs kill -9
busybox ps -ef | busybox grep -v grep | busybox egrep 'kpsmouseds' | busybox awk '{print $1}' | busybox xargs kill -9
busybox ps -ef | busybox grep -v grep | busybox egrep 'kintegrityds' | busybox awk '{print $1}' | busybox xargs kill -9
busybox ps -ef | busybox grep -v grep | busybox egrep 'khugepageds' | busybox awk '{print $1}' | busybox xargs kill -9

busybox rm -f /tmp/kthrotlds
busybox rm -f /tmp/kintegrityds
busybox rm -f /tmp/khugepageds
busybox rm -f /tmp/kpsmouseds
busybox rm -f /etc/cron.d/tomcat
busybox rm -f /etc/cron.d/root
busybox rm -f /var/spool/cron/root
busybox rm -f /var/spool/cron/crontabs/root
busybox rm -f /etc/rc.d/init.d/kthrotlds
busybox rm -f /etc/rc.d/init.d/kpsmouseds
busybox rm -f /etc/rc.d/init.d/kintegrityds
busybox rm -f /usr/sbin/kthrotlds
busybox rm -f /usr/sbin/kintegrityds
busybox rm -f /usr/sbin/kpsmouseds
busybox rm -f /etc/init.d/netdns
busybox rm -f /tmp/ld.so.preload*

ldconfig

# 再次清理异常进程
busybox ps -ef | busybox grep -v grep | busybox egrep 'ksoftirqds' | busybox awk '{print $1}' | busybox xargs kill -9
busybox ps -ef | busybox grep -v grep | busybox egrep 'kthrotlds' | busybox awk '{print $1}' | busybox xargs kill -9
busybox ps -ef | busybox grep -v grep | busybox egrep 'kpsmouseds' | busybox awk '{print $1}' | busybox xargs kill -9
busybox ps -ef | busybox grep -v grep | busybox egrep 'kintegrityds' | busybox awk '{print $1}' | busybox xargs kill -9
busybox ps -ef | busybox grep -v grep | busybox egrep 'khugepageds' | busybox awk '{print $1}' | busybox xargs kill -9

systemctl stop netdns
systemctl disable  netdns
```

清理完及时重启，如果无法重启，建立同名目录，chattr +i && mv /usr/bin/chattr /usr/bin/chattr-old

4、dubbo服务注册部分失效

访问异常，提示500，查日志发现 Failed to invoke the method 调用报错，但服务没有注册 ，查看ip查到原来只诸恶了6个服务，实际上应该有20个，部分注册失效导致这个问题。

![](/assets/20190423.png)

