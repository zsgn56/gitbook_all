## 安装 {#安装}

```
需要Python和pip 这个安装就不多说了
安装shadowsocks
pip install  shadowsocks 
这会给python的bin下面装上 ssserver(ss的服务端） sslocal(ss的代理）
在你的目录下创建shadowsocks  下面再创建标准目录 bin, conf, log

把sslocal、ssserver 拷贝到 shadowsocks/bin 下面
然后在shadowsocks/conf下面创建ss.conf 或者ss.conf 本质是个json文件 .conf是个人习惯编辑这个文件

{
"server":"你的IP",
"server_port":8888,
"local_port":1080,
"password":"1234567890",
"timeout":600,
"method":"aes-256-cfb",
"workers":16
}

然后尝试启动
/home/nick/shadowsocks/bin/ssserver -c /home/nick/shadowsocks/conf/ss.conf
```

## 运行 {#运行}

```
一般情况下，这种程序我一般都会选择使用supervisor守护进程启动
所以
pip install supervisor 安装supervisor
具体部署和ss基本一样

配置
[program:ssserver]
command=/home/nick/shadowsocks/bin/ssserver -c /home/nick/shadowsocks/conf/ss.conf
diretory=/home/nick/shadowsocks
user=nick

/home/nick/superctl update  加载配置文件。
```



