### 1、安装jdk环境

### 2、下载安装kettle

https://sourceforge.net/projects/pentaho/files/Data%20Integration/

```
 yum install libgtk
 yum install gtk2
 yun install libXtst
unzip pdi-ce-7.1.0.0-12.zip
```

### 3、安装xulrunner

否则会报错：

2017/09/25 13:52:55 - org.pentaho.di.ui.util.EnvironmentUtils@51351f28 - ERROR \(version 7.1.0.0-12, build 1 from 2017-05-16 17.18.02 by buildguy\) : Could not open a browser

2017/09/25 13:52:56 - org.pentaho.di.ui.util.EnvironmentUtils@51351f28 - ERROR \(version 7.1.0.0-12, build 1 from 2017-05-16 17.18.02 by buildguy\) : org.eclipse.swt.SWTError: No more handles \[MOZILLA\_FIVE\_HOME=''\] \(java.lang.UnsatisfiedLinkError: Could not load SWT library. Reasons:

```
wget http://ftp.mozilla.org/pub/mozilla.org/xulrunner/nightly/2012/03/2012-03-02-03-32-11-mozilla-1.9.2/xulrunner-1.9.2.28pre.en-US.linux-x86_64.tar.bz2

tar jxvf xulrunner-1.9.2.28pre.en-US.linux-x86_64.tar.bz2

cd ./xulrunner

./xulrunner -register-global
```

### 4、启动kettle

```
cd data-integration/
chmod +x -R *.sh
./kitchen.sh 
```

```
vnc登录图形界面
./spoon.sh
```



