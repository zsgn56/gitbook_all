## 基本环境 {#安装基本环境}

### 1、安装 Git 、Python-pip {#安装git}

yum install -y git python-pip

### 2、安装 NVM、Node.js、NPM {#安装nvmnodejs和npm}

\# 下载安装，安装包位置随意

wget [https://npm.taobao.org/mirrors/node/v7.2.1/node-v7.2.1-linux-x64.tar.gz](https://npm.taobao.org/mirrors/node/v7.2.1/node-v7.2.1-linux-x64.tar.gz) tar zxvf node-v7.2.1-linux-x64.tar.gz

cd  node-v7.2.1-linux-x64

\# 命令设置全局，因为安装node自带npm，所以不需要安装

sudo ln -s /home/apps/node-v7.2.1-linux-x64/bin/node /usr/local/bin/node

sudo ln -s /home/apps/node-v7.2.1-linux-x64/bin/npm /usr/local/bin/npm

\# 查看安装版本

node -v

\# 查看npm版本

npm -v

### 3、安装 gitbook {#安装nvmnodejs和npm}

npm install gitbook-cli -g

sudo ln -s /home/apps/node-v7.2.1-linux-x64/bin/gitbook /usr/local/bin/gitbook

gitbook -v

\# 安装淘宝定制的cnpm来替代npm

npm install -g cnpm --registry=[https://registry.npm.taobao.org](https://registry.npm.taobao.org) sudo

ln -s /home/apps/node-v7.2.1-linux-x64/bin/cnpm /usr/local/bin/cnpm

\# 安装gitbook和以上方式一样，只需把npm修改为cnpm

\#加入环境变量/etc/profile

export NODE\_HOME=/usr/local/node

export PATH=$NODE\_HOME/bin:$PATH

### 4、启动 gitbook {#安装nvmnodejs和npm}

cd  /home/apps/gitbook

mkdir demo

cd demo

\# 初始化之后会看到两个文件，README.md ，SUMMARY.md

gitbook init

\# 生成静态站点，当前目录会生成\_book目录，即web静态站点

gitbook build ./

\# 启动web站点，默认浏览地址：[http://localhost:4000](http://localhost:4000)

gitbook serve ./

### 5、加入到supervisor后台运行 {#安装nvmnodejs和npm}

vi /etc/supervisord.conf 加入配置

\[program:node\]

environment=NODE\_HOME=/usr/local/node

directory=/usr/local/gitbook/hk/

command=/usr/local/node/bin/gitbook serve

user=root

numprocs=1

autostart=true

autorestart=true

startretries=3

stderr\_logfile=/var/log/node.error.log

stdout\_logfile=/var/log/node.info.log

\#\#\#重新加载supervisor\#\#\#\#

supervisorctl--》update

### 6、Jenkins自动部署 {#安装nvmnodejs和npm}

新建gitbook项目，选择command over ssh，在Exec command中输入：

rm -rf  /data/docker/project/book

cd /data/docker/project/

git clone [https://user:passwd@gitlab.\*\*\*\*\*.com/gitbook/book.git](https://gitbook_jenkins:Aa1234567@gitlab.homeking365.com/gitbook/book.git)

cp -rf /data/docker/project/book/\* /usr/local/gitbook/hk/

cd /usr/local/gitbook/hk/

source /etc/profile

/usr/local/node-v9.5.0-linux-x64/bin/gitbook init

\#gitbook serve

gitlab上的gitbook项目设置webhook：

填入jenkins触发地址，动作选择push![](/assets/import.png)

