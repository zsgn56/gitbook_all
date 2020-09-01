1、centos7.4下

```
yum install ansible
```

默认可以yum，如果不行先 rpm -ivh [https://mirror.webtatic.com/yum/el7/epel-release.rpm](https://mirror.webtatic.com/yum/el7/epel-release.rpm)

2、配置SSH验证

```
#服务端
ssh-keygen

vi /etc/ansible/hosts
[test-servers]
192.168.140.56
192.168.140.57

#客户端
ssh-copy-id -i root@192.168.140.56
ssh-copy-id -i root@192.168.140.57
```

3、执行shell命令

```
ansible -m command -a "uptime" 'test-servers'       #执行uptime
ansible -m command -a "useradd mark" 'test-servers' #增加用户
```



