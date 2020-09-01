### 参考：

[https://blog.csdn.net/mingyu1016/article/details/53925611](https://blog.csdn.net/mingyu1016/article/details/53925611)

[https://www.cnblogs.com/yyhh/p/6106472.html](https://www.cnblogs.com/yyhh/p/6106472.html)

### 前提：

安装jdk、zookeeper（提供给pp-collector、pp-web）

## 安装hbase {#安装}

```
解压hbase-1.2.6-bin.tar.gz

vi /data/hbase-1.2.6/conf/hbase-site.xml

<configuration>
  <property>
        <name>hbase.zookeeper.property.clientPort</name>
    <value>2182</value>
  </property>
  <property>
        <name>hbase.rootdir</name>
        <value>/data/hbase-1.2.6/data</value>
  </property>
  <property>
        <name>zookeeper.session.timeout</name>
        <value>90000</value>
  </property>
  <property>
        <name>hbase.regionserver.restart.on.zk.expire</name>
        <value>true</value>
  </property>
</configuration>

vim log4j.properties

hbase.root.logger=INFO,console
hbase.security.logger=INFO,console
hbase.log.dir=/nas_data/data/pinpoint/hbase/logs
hbase.log.file=hbase.log


$ ../bin/start-hbase.sh        // 启动 Hbase
```

## 初始化hbase {#安装}

```
安装cronnolog
wget http://cronolog.org/download/cronolog-1.6.2.tar.gz
make install

wget  https://raw.githubusercontent.com/naver/pinpoint/master/hbase/scripts/hbase-create.hbase
./hbase shell /usr/local/src/hbase-create.hbase
完成后查看
 ./bin/hbase shell
```

## 安装pinpoint-collector {#安装}

```
unzip pinpoint-collector-1.5.2.war -C /data/tomcat-pinpoint-collector/webapps/ROOT

vi /data/tomcat-pinpoint-collector/webapps/ROOT/WEB-INF/classes/hbase.properties 
hbase.client.host=localhost
hbase.client.port=2182

vi /data/tomcat-pinpoint-collector/webapps/ROOT/WEB-INF/classes/pinpoint-collector.properties 
cluster.enable=true
cluster.zookeeper.address=localhost
cluster.zookeeper.sessiontimeout=30000
```

## 安装pinpoint-web {#安装}

```
unzip pinpoint-web-1.5.2.war  -C /data/tomcat-pinpoint-web/webapps/ROOT

修改tomcat的配置文件，防止端口号冲突

vi /data/tomcat-pinpoint-collector/webapps/ROOT/WEB-INF/classes/hbase.properties 
hbase.client.host=localhost
hbase.client.port=2182

# hbase default:/hbase
hbase.zookeeper.znode.parent=/hbase
```

## 安装pinpoint-agent {#安装}

```
将agent 拷贝到指定目录，修改日志级别
```

## 程序调用实例 {#安装}

    #定义pinpoint项目名称
    PNAME="ssps"


    #创建服务启动脚本
    cat << eof > run.sh
    #!/bin/bash
    sh /usr/local/ssps-service/sbin/start.sh
    eof

    #定义启动脚本
    cat << eof > start.sh
    #!/bin/bash
    source /etc/profile
    source ~/.bash_profile

    cd \`dirname \$0\`
    BIN_DIR=\`pwd\`
    cd ..
    DEPLOY_DIR=\`pwd\`
    CONF_DIR=\$DEPLOY_DIR/conf
    LOGS_DIR=\$DEPLOY_DIR/logs

    PINPOINT_ID=\`hostname |awk -F "-" '{print \$(NF-4)"_"\$(NF-3)"_"\$(NF-2)"_"\$NF}'|cut -c 1-24\`
    SERVER_NAME=${SERVICE_NAME}
    SERVER_PORT=${DUBBO_PORT}
    STDOUT_FILE=\$LOGS_DIR/stdout.log

    if [ -z "\$SERVER_NAME" ]; then
        SERVER_NAME=\`hostname\`
    fi

    if [ ! -d \$LOGS_DIR ]; then
        mkdir \$LOGS_DIR
    fi

    if [ ! -d \$LOGS_DIR/dump ]; then
        mkdir -p \$LOGS_DIR/dump
    fi

    PIDS=\`ps -ef | grep java | grep "\$CONF_DIR" |awk '{print \$2}'\`
    if [ -n "\$PIDS" ]; then
        echo "ERROR: The \$SERVER_NAME already started!"
        echo "PID: \$PIDS"
        exit 1
    fi

    if [ -n "\$SERVER_PORT" ]; then
        SERVER_PORT_COUNT=\`netstat -tln | grep \$SERVER_PORT | wc -l\`
        if [ \$SERVER_PORT_COUNT -gt 0 ]; then
            echo "ERROR: The \$SERVER_NAME port \$SERVER_PORT already used!"
            exit 1
        fi
    fi


    LIB_DIR=\$DEPLOY_DIR/lib
    LIB_JARS=\`ls \$LIB_DIR|grep .jar|awk '{print "'\$LIB_DIR'/"\$0}'|tr "\n" ":"\`

    JAVA_OPTS=" -Djava.awt.headless=true -Djava.net.preferIPv4Stack=true "
    JAVA_DEBUG_OPTS="-XX:HeapDumpPath=\$LOGS_DIR/dump -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+HeapDumpOnOutOfMemoryError  -Xloggc:\$LOGS_DIR/dump/gc.log"
    if [ "\$1" = "debug" ]; then
        JAVA_DEBUG_OPTS=" -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n "
    fi
    JAVA_JMX_OPTS=""
    if [ "\$1" = "jmx" ]; then
        JAVA_JMX_OPTS=" -Dcom.sun.management.jmxremote.port=1099 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false "
    fi
    JAVA_APM_OPTS=" -javaagent:/usr/local/pinpoint_agent/pinpoint-bootstrap-1.7.0-RC2.jar -Dpinpoint.agentId=\$PINPOINT_ID -Dpinpoint.applicationName=${PNAME} "
    if [ "\$1" = "apm" ]; then
        JAVA_APM_OPTS=" -javaagent:/usr/local/pinpoint_agent/pinpoint-bootstrap-1.7.0-RC2.jar -Dpinpoint.agentId=\$PINPOINT_ID -Dpinpoint.applicationName=${PNAME} "
    fi
    JAVA_MEM_OPTS=""
    BITS=\`java -version 2>&1 | grep -i 64-bit\`
    if [ -n "\$BITS" ]; then
        JAVA_MEM_OPTS=" -server -Xmx2048m -Xms2048m -Xmn512m -XX:PermSize=384m -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 "
    else
        JAVA_MEM_OPTS=" -server -Xmx2048m -Xms2048m -XX:PermSize=384m -XX:SurvivorRatio=2 -XX:+UseParallelGC "
    fi

    echo -e "Starting the \$SERVER_NAME ...\c"
    nohup java \$JAVA_OPTS \$JAVA_MEM_OPTS \$JAVA_DEBUG_OPTS \$JAVA_JMX_OPTS \$JAVA_APM_OPTS -Ddubbo.reference.check=false -classpath \$LIB_JARS -Ddubbo.spring.config=classpath*:applicationContext.xml com.alibaba.dubbo.container.Main > \$STDOUT_FILE 2>&1 &

    COUNT=0
    while [ \$COUNT -lt 1 ]; do    
        echo -e ".\c"
        sleep 1 
        if [ -n "\$SERVER_PORT" ]; then
            if [ "\$SERVER_PROTOCOL" == "dubbo" ]; then
                COUNT=\`echo status | nc -i 1 127.0.0.1 \$SERVER_PORT | grep -c OK\`
            else
                COUNT=\`netstat -an | grep \$SERVER_PORT | wc -l\`
            fi
        else
            COUNT=\`ps -f | grep java | grep "\$DEPLOY_DIR" | awk '{print \$2}' | wc -l\`
        fi
        if [ \$COUNT -gt 0 ]; then
            break
        fi
    done

    echo "OK!"
    PIDS=\`ps -f | grep java | grep "\$DEPLOY_DIR" | awk '{print \$2}'\`
    echo "PID: \$PIDS"
    echo "STDOUT: \$STDOUT_FILE"
    eof

编辑项目启动脚本

vi start.sh

:%s\#\`\#\\`\#g

:%s\#$\#$\#g

修改位置：

    1、
    PINPOINT_ID=\`hostname |awk -F "-" '{print \$(NF-4)"_"\$(NF-3)"_"\$(NF-2)"_"\$NF}'|cut -c 1-24\`
    2、
    JAVA_APM_OPTS=" -javaagent:/usr/local/pinpoint_agent/pinpoint-bootstrap-1.7.0-RC2.jar -Dpinpoint.agentId=\$PINPOINT_ID -Dpinpoint.applicationName=${PNAME} "
    if [ "\$1" = "apm" ]; then
        JAVA_APM_OPTS=" -javaagent:/usr/local/pinpoint_agent/pinpoint-bootstrap-1.7.0-RC2.jar -Dpinpoint.agentId=\$PINPOINT_ID -Dpinpoint.applicationName=${PNAME} "
    fi
    3、
    nohup java \$JAVA_OPTS \$JAVA_MEM_OPTS \$JAVA_DEBUG_OPTS \$JAVA_JMX_OPTS \$JAVA_APM_OPTS



