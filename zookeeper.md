1、迁移data目录后日志报错：

```
efusing session request for client /127.0.0.1:48430 as it has seen zxid 0x26eea7 our last zxid is 0x2a client
```

解决方法：重启所有客户端\(例如用到zookeeper的web项目\).



2、空间满导致注册的服务重复：

```
 ./zkCli.sh -server 192.168.100.157
 ls /dubbo/com.hk.ums.api.push.IServiceAppPushService/providers
```

解决方法：

```
/usr/local/zookeeper/bin/zkServer.sh stop
mv /data/zookeeper /data/zookeeperold
mkdir -p /data/zookeeper/data
echo 0 > /data/zookeeper/data/myid
/usr/local/zookeeper/bin/zkServer.sh start
确认正常后，重启dubbo
```



