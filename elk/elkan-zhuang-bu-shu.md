# ELK+Filebeat+Kafka+ZooKeeper

性能：30分钟内可插入500w条记录

#### 1、硬件及系统配置

总共5台服务器：

3台ES 4核32G 其中1台部署logstash（ input kafka，json转换）

2台2核8G zookeeper+kafka+logstash  （ouput kafka，处理切割字段）

vim /etc/security/limits.conf

ulimit -SHn 65536

sysctl -w vm.max\_map\_count=262144

sysctl -p

vi /etc/security/limits.d/90-nproc.conf

\* soft nofile 65536

#### 2、安装elasticsearch

调整elasticsearch中jvm内存

groupadd elasticsearch

useradd elasticsearch -g elasticsearch

chown -R elasticsearch.elasticsearch  -R /usr/local/elasticsearch

su elasticsearch

cd /usr/local/elasticsearch

./bin/elasticsearch &

#### 3、安装zookeeper

vi  /user/local/zookeeper/conf/zoo.cfg

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/logs
clientPort=2181
maxClientCnxns=5000


server.1=192.168.10.2:2887:3887
server.2=192.168.10.3:2887:3887
server.3=192.168.10.4:2887:3887
```

echo 1 &gt;/data/zookeeper/data/myid \(分别在3台上输入不同的id）

/usr/local/zookeeper/bin/zkServer.sh start

/usr/local/zookeeper/bin/zkServer.sh status

/usr/local/zookeeper/bin/zkCli.sh -server localhost:2181

ls /brokers/ids 查询brokder注册id，ls /brokers/topics 查询topic

#### 4、安装kafka

vim  /usr/local/kafka/config/server.properties

```
broker.id=0
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/usr/local/kafka/logs
num.partitions=1
num.recovery.threads.per.data.dir=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
log.retention.hours=24
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=192.168.10.2:2181,192.168.10.3:2181,192.168.10.4:2181
zookeeper.connection.timeout.ms=6000
group.initial.rebalance.delay.ms=0
delete.topic.enable=true
```

```
修改传输最大值
vi service.cfg

#broker能接收消息的最大字节数
message.max.bytes=20000000
#broker可复制的消息的最大字节数
fetch.message.max.bytes=2000000
replica.fetch.max.bytes=2000000
max.request.sizes=20000000

vi producer.properties
max.request.size=20000000

vi consumer.properties
max.partition.fetch.bytes=20000000
```

vi /etc/hosts 加入各台主机名,修改broker.id

/usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties

/usr/local/kafka/bin/kafka-topics.sh --create  --zookeeper 192.168.10.75:2181 --replication-factor 2 --partitions 2 --topic log

/usr/local/kafka/bin/kafka-topics.sh --delete --topic log --zookeeper 192.168.200.24:2181/kafka

/usr/local/kafka/bin/kafka-topics.sh  --describe --zookeeper 192.168.10.75:2181

```
Topic:log    PartitionCount:2    ReplicationFactor:2    Configs:
    Topic: log    Partition: 0    Leader: 1    Replicas: 1,0    Isr: 1,0
    Topic: log    Partition: 1    Leader: 0    Replicas: 0,1    Isr: 0,
```

/usr/local/kafka/bin/kafka-topics.sh --list --zookeeper 192.168.10.75:2181

/usr/local/kafka/bin/kafka-consumer-groups.sh --describe --group logstash --bootstrap-server 192.168.10.76:9092

```
TOPIC                          PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG        CONSUMER-ID                                       HOST                           CLIENT-ID
kflog                          0          28142           28898           756        -                                                 -                              -
                                          已消费           总数量       未消费
```

#### 5、安装logstash（input filebeat）

vim /usr/local/logstash/logstash.conf

```
input { 
  beats {
    port => 5044
     }  
 }

output {
  kafka {
    bootstrap_servers => "192.168.10.76:9092"
    topic_id => "kflog"
    codec => "json"
   batch_size => "2000000"
   max_request_size => "2000000"
 }
```

/usr/local/logstash/bin/logstash -f /usr/local/logstash/logstash.conf --config.reload.automatic （--verbose --debug 调试加）

#### 6、安装 filebeat

vim /usr/local/logstash/logstash.conf

```
filebeat.prospectors:

- type: log
  paths:
    - /logs/service-130/173/service-abs/abs.log
    - /logs/service-130/174/service-abs/abs.log
  multiline.pattern: ^[0-9]{4}-[0-9]{2}-[0-9]{2}
  multiline.negate: true
  multiline.match: after

  fields:
      project: service-abs

- type: log
  paths:
    - /logs/service-130/171/service-acs/acs.log
    - /logs/service-130/172/service-acs/acs.log
    - /logs/service-130/173/service-acs/acs.log
    - /logs/service-130/174/service-acs/acs.log
  multiline.pattern: ^[0-9]{4}-[0-9]{2}-[0-9]{2}
  multiline.negate: true
  multiline.match: after

  fields:
      project: service-acs


output:
  logstash:
    hosts: ["192.168.10.76:5044"]
```

vim /usr/local/filebeat/config/filebeat.yml 启用监控

```
xpack.monitoring:
  enabled: true
  elasticsearch:
    hosts: ["http://192.168.200.27:9200"]
```

/usr/local/filebeat/filebeat -c /usr/local/filebeat/filebeat.yml &

```
注:paths 中采用/logs/service-130/*/service-acs/acs.log方式，日志存放在节点本地，nfs上将节点日志目录挂载群集目录，
   在维护好日志目录的情况下，可以保证弹性伸缩服务器节点时，日志收取正常
```

#### 7、安装logstash（ouput es）

vim /usr/local/filebeat/filebeat.yml

```
input{
    kafka {
        bootstrap_servers => "192.168.10.75:9092,192.168.10.76:9092"
        topics => ["log"]
        consumer_threads => 5
        decorate_events => true

}

}

filter {
        json {
            source => "message"
            #target => "doc"
            #remove_field => ["message"]
        }
}

output {
  # 输出到屏幕
  # stdout {codec => rubydebug}
  # 输出到elasticsearch
  elasticsearch {
     hosts => "192.168.30.4:9200"
     index => "logstash-%{[fields][project]}-%{+YYYY.MM.dd}"

   }
}
```

/usr/local/logstash/bin/logstash -f /usr/local/logstash/logstash.conf --config.reload.automatic

logstash.yml启用monitor监控

```
xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.url: ["http://192.168.200.27:9200"]
```

#### 8、安装kibana

vim /usr/local/kibana/config/kibana.yml

```
server.host: "localhost"
elasticsearch.url: "http://localhost:9200"
```

/usr/local/kibana/bin/kibana &

