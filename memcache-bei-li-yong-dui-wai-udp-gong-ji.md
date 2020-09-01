问题：查看账单异常，机器流量异常

iptraf-nlog查看流量异常，

![](/assets/202002121)

iftop 查看耗流量的多个未知ip

![](/assets/202002122.png)

* iptraf-ng和tcpdump查看UDP请求异常
* ![](/assets/2020021203.png) 
* tcpdump -n udp port 80 查看对外攻击

![](/assets/2020021204.png)

至此排查出是memcache的问题：

1、当时配置时候开放了公网连接

2、没有鉴权

处理：配置memchache只允许那内网连接，并在防火墙上限制连接

