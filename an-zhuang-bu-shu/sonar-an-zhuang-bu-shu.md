## 参考 {#安装}

[https://blog.csdn.net/yuan\_xw/article/details/54767182](https://blog.csdn.net/yuan_xw/article/details/54767182)

[https://blog.csdn.net/csolo/article/details/78159521](https://blog.csdn.net/csolo/article/details/78159521)

## 创建mysql数据库 {#安装}

```
CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;

CREATE USER 'sonar' IDENTIFIED BY 'sonarqube';

GRANT ALL ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'sonarqube';

GRANT ALL ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonarqube';
```

## 下载解压sonar后配置 {#安装}

```
vim /usr/local/sonar/conf/sonar.properties

sonar.jdbc.username=sonar
sonar.jdbc.password=sonarqube
sonar.jdbc.url=jdbc:mysql://192.168.100.206:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
```

## 安装中文 {#安装}

```
下载地址：https://github.com/SonarQubeCommunity/sonar-l10n-zh/releases/，找到对应的tag

git clone https://github.com/SonarQubeCommunity/sonar-l10n-zh.git
git tag
git checkout sonar-l10n-zh-plugin-1.19
mvn install

cp sonar-l10n-zh-plugin-1.19-RC2-SNAPSHOT.jar /usr/local/sonarqube/extensions/plugins
```

## 创建用户启动 {#安装}

```
 groupadd sonar
 useradd sonar -g sonar
 chown -R sonar:sonar /usr/local/sonarqube

 su sonar
 /usr/local/sonarqube/bin/linux-x86-64/sonar.sh start
```

异常关闭的的导致启动失败后的处理：删除es数据目录上的节点数据。

## mvn项目使用 {#安装}

```
192.168.100.18:9000 admin admin
生产token sonar: c1efaba1849f890019eb9e8c442e3ce901ae8d88

使用Maven执行SonarQube扫描

mvn sonar:sonar \
  -Dsonar.host.url=http://192.168.100.18:9000 \
  -Dsonar.login=c1efaba1849f890019eb9e8c442e3ce901ae8d88
```

## Jenkins项目使用 {#安装}

1、自动安装插件或者手动安装插件

jenkins -》插件管理-》高级

![](/assets/2018731.png)

2、配置 sonarqube server

jenkins-》系统管理-》系统配置，输入地址和token![](/assets/20187312.png)3、安装配置sonar scanner

jenkins-》系统管理-》Global Tool Configuration![](/assets/20187313.png)

4、配置项目

sonar.projectKey=service-cps

sonar.projectName=service-cps

sonar.projectVersion=0.1

sonar.modules=hk-cps-service-api,hk-cps-service

sonar.branch=qa

sonar.sourceEncoding=UTF-8

sonar.language=java

sonar.sources=src/main/java

sonar.tests=src/test/java

sonar.java.binaries=./

\#排除一些不想统计的类

sonar.java.coveragePlugin=jacoco

sonar.junit.reportsPath=target/surefire-reports

sonar.surefire.reportsPath=target/surefire-reports

5、jacoco覆盖率

在mvn命令中增加 org.jacoco:jacoco-maven-plugin:0.7.9:prepare-agent

