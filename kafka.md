\[root@elk-75 logs\]\# /usr/local/kafka/bin/kafka-consumer-groups.sh --describe --group logstash --bootstrap-server 192.168.10.75:9092

Note: This will not show information about old Zookeeper-based consumers.

Error: Consumer group 'logstash' does not exist.

消费者服务宕机超过1天后提示，无法找到，生产者队列查看一直产生数据，但重启消费者服务后队列显示信息都被消费了，实际并没有。

愿意未知

怀疑被offset被直接重置：

\[2018-09-10 21:48:30,409\] WARN Found a corrupted index file due to requirement failed: Corrupt index found, index file \(/data/kafka-logs/klog-1/00000000000150556624.index\) has non-zero size but the last offset is 150556624 which is no larger than the base offset 150556624.}. deleting /data/kafka-logs/klog-1/00000000000150556624.timeindex, /data/kafka-logs/klog-1/00000000000150556624.index, and /data/kafka-logs/klog-1/00000000000150556624.txnindex and rebuilding index... \(kafka.log.Log\)

当前存在问题：

1、logstash容易故障，input和output存在性能问题

2、kafka数据丢失

3、filebeat收集数据不全



