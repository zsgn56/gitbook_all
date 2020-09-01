参考 [https://www.cnblogs.com/luotaoyeah/p/3791966.html](https://www.cnblogs.com/luotaoyeah/p/3791966.html)

![](/assets/2018.8.20nexus.png)![](/assets/2018.8.20.1nexus.png)Nexus预定义了3个本地仓库，分别是Releases, Snapshots, 3rd Party. 

**　　Releases:**

 　　　　这里存放我们自己项目中发布的构建, 通常是Release版本的 

**　　Snapshots:**

 　　　　这个仓库非常的有用, 它的目的是让我们可以发布那些非release版本, 非稳定版本

**　　3rd Party:**

　　 第三方库, 你可能会问不是有中央仓库来管理第三方库嘛,没错, 这里的是指可以让你添加自己的第三方库, 比如有些构件在中央仓库是不存在的. 比如你在中央仓库找不到[Oracle](http://lib.csdn.net/base/oracle)的JDBC驱动, 这个时候我们就需要自己添加到3rdparty仓库

