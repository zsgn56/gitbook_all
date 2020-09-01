### 1、安装VNC

```
yum install -y tigervnc tigervnc-server
```

### 2、安装安装X-Window

```
# yum check-update
# yum groupinstall "X Window System"
# yum install gnome-classic-session gnome-terminal nautilus-open-terminal control-center liberation-mono-fonts
# unlink /etc/systemd/system/default.target
# ln -sf /lib/systemd/system/graphical.target /etc/systemd/system/default.target
# reboot
```

### 3、从VNC备份库中复制service文件到系统service服务管理目录下

```
# cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
```

### 4、修改vncserver@:1.service文件

```
[Unit]
Description=Remote desktop service (VNC)
After=syslog.target network.target

[Service]
Type=forking
user=root

ExecStartPre=-/usr/bin/vncserver -kill %i
ExecStart=/sbin/runuser -l root -c "/usr/bin/vncserver %i" 
PIDFile=/root/.vnc/%H%i.pid
ExecStop=-/usr/bin/vncserver -kill %i

[Install]
WantedBy=multi-user.targe
```

修改文件使配置生效：

```
# systemctl daemon-reload
```

### 5、为vncserver@:1.service设置密码

```
# vncpasswd
```

### 6、启动VNC

```
# systemctl enable vncserver@:1.service 

# systemctl start vncserver@:1.service #启动vnc会话服务

# systemctl status vncserver@:1.service 

# systemctl stop vncserver@:1.service #关闭nvc会话服务

# netstat -lnt | grep 590*     

tcp        0      0 0.0.0.0:5901            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:5901            0.0.0.0:*               LISTEN

# yum whatprovides "*/xhost"
# xhost +
```

### 8、windows使用VNC链接到图形化界面

1、链接图形化界面服务器的5901端口  
![](http://i2.51cto.com/images/blog/201801/15/2dfe3a15de542ade810005acdf244436.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk= "CentOS7.2安装VNC，让Windows远程连接CentOS 7.2 图形化界面")





