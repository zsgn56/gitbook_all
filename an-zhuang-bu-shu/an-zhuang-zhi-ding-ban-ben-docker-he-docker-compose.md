```
参考：
https://www.cnblogs.com/lonelyxmas/p/10430207.html
https://github.com/docker/compose/releases/
```

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

安装最新版本或者指定版本docker

```
yum install docker-ce -y
yum install docker-ce-18.09.2-3.el7
```

安装指定版本docker-compose

    curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose

```
pip install --upgrade pip
pip uninstall urllib3
pip uninstall -y chardet
sudo pip install requests

```



