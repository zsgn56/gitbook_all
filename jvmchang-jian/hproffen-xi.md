1、outofmemory error分析

参考：[https://blog.csdn.net/liuhaiabc/article/details/53171434](https://blog.csdn.net/liuhaiabc/article/details/53171434)

下载Eclipse Memory Analyze Tool

其中在init中的xmn要配置成hprof容器的2倍，否则启动报错

分析得出java.util.date 每个占24字节，总共有700917070个

![](/assets/hprof.png)

