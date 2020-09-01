1、输出dump


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

    SERVER_NAME=service-oms
    SERVER_PORT=28080
    STDOUT_FILE=\$LOGS_DIR/stdout.log

    if [ -z "\$SERVER_NAME" ]; then
        SERVER_NAME=\`hostname\`
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
    JAVA_JVMS=" -XX:HeapDumpPath=\$LOGS_DIR/dump -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+HeapDumpOnOutOfMemoryError  -Xloggc:\$LOGS_DIR/dump/heap_trace.txt "
    JAVA_DEBUG_OPTS=""
    if [ "\$1" = "debug" ]; then
        JAVA_DEBUG_OPTS=" -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n "
    fi
    JAVA_JMX_OPTS=""
    if [ "\$1" = "jmx" ]; then
        JAVA_JMX_OPTS=" -Dcom.sun.management.jmxremote.port=1099 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false "
    fi
    JAVA_MEM_OPTS=""
    BITS=\`java -version 2>&1 | grep -i 64-bit\`
    if [ -n "\$BITS" ]; then
        JAVA_MEM_OPTS=" -server -Xmx800m -Xms800m -Xmn256m -XX:PermSize=384m -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 "
    else
        JAVA_MEM_OPTS=" -server -Xmx800m -Xms800m -XX:PermSize=384m -XX:SurvivorRatio=2 -XX:+UseParallelGC "
    fi

    echo -e "Starting the \$SERVER_NAME ...\c"
    nohup java \$JAVA_OPTS \$JAVA_MEM_OPTS \$JAVA_DEBUG_OPTS \$JAVA_JMX_OPTS \$JAVA_JVMS -classpath \$CONF_DIR:\$LIB_JARS com.alibaba.dubbo.container.Main > \$STDOUT_FILE 2>&1 &

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



2、包含pinpoint

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
    JAVA_DEBUG_OPTS=""
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
        JAVA_MEM_OPTS=" -server -Xmx800m -Xms800m -Xmn256m -XX:PermSize=384m -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 "
    else
        JAVA_MEM_OPTS=" -server -Xmx800m -Xms800m -XX:PermSize=384m -XX:SurvivorRatio=2 -XX:+UseParallelGC "
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



