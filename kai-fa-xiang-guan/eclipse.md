### 1、eclipse引入jar包

下面看第一种方式：基本步骤式

```
右键项目属性，选择Property，在弹出的对话框左侧列表中选择Java Build Path，如下图所示：选择Add External JARs，就可以逐个（也可以选择多个，但是限制在同一个文件夹中）添加第三方引用jar包。
```

![](http://img307.ph.126.net/CV8LXav_jD9YKnYHc5udPA==/3796815960850206160.jpg)

上面这种方式的操作具有通用性，但是每次创建项目，都需要重新引入Jar包，Jar包不具有可重用性

第三种方式：文件夹导入式

```
在项目中，创建新的文件夹，如下图所示，本示例中创建了structs\_jar文件夹 ，将第三方的jar包拷贝到该文件夹中。
```

![](http://img.ph.126.net/BsP-DQSXDB7Q0mKgWM0mkg==/1123648107045760605.jpg)

```
选中需要添加的jar包，然后右键选择"Build PathàAdd to Build Path"，这样Jar包就导入成功了。
```

![](http://img.ph.126.net/H5SDJROyFvqwhNLhyJQg8g==/1297599642652945826.jpg)

第三种导入Jar包的方式比第二种更简单，而且重用性更强，当我们不同的机器上查找所需的Jar包时，将这些文件夹直接拷贝就可以。工具决定效率，方法决定效率，应该采用灵活的方式去使用工具。

### 2、eclipse支持spring

1、在线安装Spring的jar包点击help，选择Eclipse Marketplace选项

输入Spring Tool

点击Install 进行安装，图中我的是已经安装好的了

2、下载下来spring-framework-4.3.4.RELEASE-dist包

如果这个包里没有commons-logging的包那就要单独把这个也下载下来commons-logging-1.2-bin.tar

解压后的导入jar包

3、项目pom.xml引入项目

### 3、spring的applicationContext路径

两种方式，相对路径和绝对路径

相对路径根目录 src/main/java

### 4、Eclipse打包可执行的jar包

1. Expoart -&gt; java -&gt; Runnable JAR file
2. Launch configuration：选择main方法所在的文件/类
3. Export destination：选择或填写jar包的名字，如：d:\a.jar
4. Library Handling：建议选择第二种
5. 点击“Finish”按钮，生成jar文件

### 5、Eclipse引入外部maven仓库项目

%M2\_HOME%/conifg/settings.xml 中找到仓库目录，将外部的maven的仓库复制到本地，选择reindex

（有新更新时也是如此操作）

![](/assets/2018.8.30.png)

