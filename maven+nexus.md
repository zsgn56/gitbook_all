### 1、项目compass在jenkins上maven构建报警，jar提示有错

```
WARNING] The POM for com.hk.egg:egg-common:jar:1.2.1-20180403.020849-55 is invalid, transitive dependencies (if any) will not be available, enable debug logging for more details
```

原因：依赖项目egg没有更新。

处理过程：进入项目workspace，执行mvn dependency:tree&gt;tree.txt，查找error

![](/assets/egg-error.png)

清空旧的依赖jar包 rm -rf /data/jenkins/maven/repo/com/hk/egg

清空工作空间 rm -rf /data/jenkins/jobs/service-compass/workspace/\*

重新构建egg

重新构建compass

### 2、mvn命令中 org.jacoco:jacoco-maven-plugin:0.7.9:prepare-agent 分别指groupname:plugin-name：goal-name

### 3、-Dmaven.test.skip=true  排除测试，如果要测试覆盖率，不能加这个选项

#### 4、使用install:file命令安装jar到maven本地仓库

报错：Failed to execute goal on project sentinel-spring-boot-starter: Could not resolve dependencies for project com.alibaba.csp:sentinel-spring-boot-starter:jar:0.1.1-SNAPSHOT

原因：本地mvn缺乏jar

\#mvn install:install-file -DgroupId=com.alibaba.csp -DartifactId=sentinel-core -Dversion=0.1.1-SNAPSHOT -Dpackaging=jar -Dfile=sentinel-core-0.1.1-SNAPSHOT.jar

或者mvn clean  install 相关jar源程序

### 5、error assembling JAR: For artifact {junit:junit:null:jar}: The version cannot be empty.

pom.xml中的junit需要指定版本号。而且为4.\*

### 6、maven项目引用外部jar包的方法

[https://www.jb51.net/article/130618.htm](https://www.jb51.net/article/130618.htm)

pom.xml 修改

```
    <properties>
        <sentinel.version>0.1.1-SNAPSHOT</sentinel.version>
    </properties>
        <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.alibaba.csp</groupId>
                <artifactId>sentinel-core</artifactId>
                <version>${sentinel.version}</version>
            </dependency>
```

子目录中pom.xml 修改

```
    <dependencies>
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-core</artifactId>
        </dependency>
```

### 7、mvn deploy提示401

```
 mvn中的setting.XML
   <servers>
        <server>
          <id>snapshots</id>
          <username>admin</username>
          <password>admin123</password>
        </server>
  </servers>
```

```
项目中的pom.xml

    <distributionManagement>
        <repository>
            <id>releases</id>
            <name>Team Nexus Release Repository</name>
            <url>http://192.168.100.82:8081/nexus/content/repositories/releases</url>
        </repository>
        <snapshotRepository>
            <id>snapshots</id>
            <name>Team Nexus Snapshot Repository</name>
            <url>http://192.168.100.82:8081/nexus/content/repositories/snapshots</url>
            <uniqueVersion>false</uniqueVersion>
        </snapshotRepository>
    </distributionManagement>
```

### 8、mvn 构建无法从中央仓库获取jar包![](/assets/maven2018.9.25.png)

原因：本地仓库中的文件或者目录权限有异常，导致冲突后无法下载

解决办法：删除本地仓库中的文件，重新构建

