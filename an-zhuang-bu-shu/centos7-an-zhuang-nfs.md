##### 参考：[https://www.cnblogs.com/st-jun/p/7742560.html](https://www.cnblogs.com/st-jun/p/7742560.html)

### 一、安装

```
 yum -y install nfs-utils
```

### 二、配置端口和目录

```
# vim /etc/sysconfig/nfs
#追加端口配置
MOUNTD_PORT=4001　　
STATD_PORT=4002
LOCKD_TCPPORT=4003
LOCKD_UDPPORT=4003
RQUOTAD_PORT=4004
```

```
 vim /etc/exports　　
/var/nfs    192.168.1.0/24(rw)

exportfs -r　　#重载exports配置
exportfs -v　　#查看共享参数
/var/nfs          192.168.1.0/24(rw,sync,wdelay,hide,no_subtree_check,sec=sys,secure,root_squash,no_all_squash)
```

### 三、启动服务

```
systemctl start rpcbind.service
systemctl enable rpcbind.service
systemctl start nfs.service
systemctl enable nfs.service
showmount -e 192.168.200.7
```



