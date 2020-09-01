参考：[https://zh.wikibooks.org/zh-hans/Shadowsocks](https://zh.wikibooks.org/zh-hans/Shadowsocks)

1、安装服务器端

```
pip install shadowsociks

启动
/usr/bin/python /usr/bin/ssserver -p 2433 -k password -m rc4-md5
```

2、安装客户端

下载软件wget [http://www.huzs.net/soft/shadowsocks-python/shadowsocks-1.3.3.tar.gz](http://www.huzs.net/soft/shadowsocks-python/shadowsocks-1.3.3.tar.gz)

```
cd /usr/local/src/shadowsocks

vi config.json

{
        "server":"147.9.51.2",
        "server_port":2433,
        "local_port":18888,
        "password":"password",
        "method": "rc4-md5",
        "timeout":600
}

nohup ./shadowsocks-local &
```



