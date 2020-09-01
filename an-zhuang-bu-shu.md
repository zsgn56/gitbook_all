

参考：[https://www.cnblogs.com/lonelyxmas/p/10430207.html](https://www.cnblogs.com/lonelyxmas/p/10430207.html)

软件包安装

```
yum install -y yum-utils  device-mapper-persistent-data lvm2
```

添加yum源

```
yum-config-manager \
--add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

查看可安装的版本

```
yum list docker-ce --showduplicates | sort -r
```

安装最新或者指定版本docker

```
yum install docker-ce -y
```

安装指定版本docker-compose

```
pip install --upgrade pip
sudo pip uninstall urllib3
sudo pip uninstall chardet

然后，再安装
sudo pip uninstall requests
pip install docker-compose==1.24.0
```



