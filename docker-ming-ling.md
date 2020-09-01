```
docker save 45945e596f6e > nexus.tar
```

```
docker load < nexus.tar

docker tag 45945e596f6e nexus:20170909

docker run -t -i nginx1.12.1 /bin/bash

docker commit b16d41b250bd nginx1.12.1-hk

docker run -d -p 8082:8081 —name app\_nexus 45945e596f6e

docker cp 0ebac92508da:/opt/app/logs/ccs-service /logs/172\_old



docker inspect - -format=’{{.NetworkSettings.IPAddress}}’ $\(docker ps -a -q\)  
在宿主机上查看容器IP

1、服务器宕机后docker无法启动解决  
删除/data/docker/docker.pid  
确认/etc/docker/daemon.json正常

2、查看docker dns

参考: http://dockone.io/article/2316

容器中的DNS名称解析优先级顺序为：

* ​ 内置DNS服务器127.0.0.11。
* ​ 通过--dns等参数为容器配置的DNS服务器。
* ​ docker守护进程的--dns服务配置（默认为8.8.8.8和8.8.4.4）
* ​ 宿主机上的DNS设置

docker inspect -f "{{.State.Pid}}" d4650d3966fb

mkdir /var/run/netns/

ln -s /proc/16594//ns/net /var/run/netns/d4650d3966fb

ip netns exec d4650d3966fb iptables-save

ip netns exec d4650d3966fb netstat -ntpl


```



