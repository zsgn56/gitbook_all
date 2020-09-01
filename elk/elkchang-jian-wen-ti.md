# 1、群集状态显示YELLOW

## 原因

yellow表示所有主分片可用，但不是所有副本分片都可用，最常见的情景是单节点时，由于es默认是有1个副本，主分片和副本不能在同一个节点上，所以副本就是未分配unassigned

## 处理

### p、r分片都为unassigned

* 过滤查看所有未分配索引的方式，第一列表示索引名，第二列表示分片编号，第三列p是主分片，r是副本

```
curl -s "http://{ESIP}:9200/_cat/shards" | grep UNASSIGNED
eslog1                 3 p UNASSIGNED
eslog1                 3 r UNASSIGNED
```

知道哪个索引的哪个分片就开始手动修复，通过reroute的allocate分配

```
curl -XPOST '{ESIP}:9200/_cluster/reroute' -d '{
    "commands" : [ {
          "allocate" : {
              "index" : "eslog1",
              "shard" : 4,
              "node" : "es1",
              "allow_primary" : true
          }
        }
    ]
}'
```

* 分配副本时必须要带参数
  `"allow_primary" : true`
  , 不然会报错
* 当集群中es版本不同时，如果这个未分配的分片是高版本生成的，不能分配到低版本节点上，反过来低版本的分片可以分配给高版本，如果遇到了，只要升级低版本节点的ES版本即可

### 只有r分片都unassigned

```
curl -s "http://10.19.22.142:9200/_cat/shards" | grep UNASSIGNED
 logstash-nginx-weixin-out-2018.04.05          2 r UNASSIGNED
```

primary shards正常，强制删除掉replica shards，设置副本为0，然后恢复为1让es再重新生成

```
curl -XPUT '192.168.30.4:9200/logstash-nginx-weixin-out-2018.04.05/_settings' -d ' { "index" : { "number_of_replicas" : 0 } }'
```

```
curl -XPUT '192.168.30.4:9200/logstash-nginx-weixin-out-2018.04.05/_settings' -d ' { "index" : { "number_of_replicas" : 1 } }'
```

# 2、logstash input kakfa失败

logstash input  kakfa没有报错，但是数据没有打出

## 原因：

收集用的logstash本地上没有做hosts，导致kafka主机的主机无法解析出ip，即使在配置文件中指定了ip也不能正常。

# 3、日志插入es失败

报错：whose UTF8 encoding is longer than the max length 32766\), all of which were skipped

## 原因：

ES对字节长度大于32766的字段串会报警

参考 [https://stackoverflow.com/questions/24019868/utf8-encoding-is-longer-than-the-max-length-32766](https://stackoverflow.com/questions/24019868/utf8-encoding-is-longer-than-the-max-length-32766)

# 4、kafka问题

用于消费的logstash停止服务，超过1-2天后，发现队列里面原来堆积的消息被清空了，包括最新产生的，

原因未知。

# 5、es节点磁盘后写入失败后导致的问题

磁盘满后无法写入，进行扩容，扩容后进行修复提示

marked shard as initializing, but shard state is \[POST\_RECOVERY\], mark shard as started\]

修复到100%后，此时master节点为其他节点，es无法写入。

后续重新停止服务，将master恢复到原来的master节点上，重新等待修复。

在同样的报错下，发现es可以写入，原因未知。

新的异常：

kafka的数据到达一定数据后，发现队列被清空了，又不像是被消费了,filebeat发现有两三天的数据写入到logstash，当时的数据早被重命名了，原因还是未知。

filebeat会写入几天前的日志，有些索引延迟建立，判断还是需要优化索引。

每天创建索引时候容易出错，参考[https://discuss.elastic.co/t/elasticsearch-2-2-0-i-am-occasionally-getting-process-cluster-event-timeout-exception-failed-to-process-cluster-event-put-mapping-as-within-30s-while-bulk-indexing-documents/42305/5](https://discuss.elastic.co/t/elasticsearch-2-2-0-i-am-occasionally-getting-process-cluster-event-timeout-exception-failed-to-process-cluster-event-put-mapping-as-within-30s-while-bulk-indexing-documents/42305/5)

经典优化：[https://blog.csdn.net/jiao\_fuyou/article/details/78487659](https://blog.csdn.net/jiao_fuyou/article/details/78487659)

重启所有服务，包括logstash、filebeat、kafka后报错明显减少。

2019.2.18 更新：

磁盘满后索引会被设置为只读，清理旧索引，释放空间重启es，等状态正常后在kibana中开启

```
PUT _settings
{
  "index": {
    "blocks": {
      "read_only_allow_delete": "false"
    }
  }
}
```

### 6、logstash 总结

总结：

1、当你无法确认所接受到的数据都能在你的自定义正则匹配时，建议删除grok插件自带的

2、写正则表达式时，多思考，要兼容意外情况。

3、默认@timestamp 为filebeat采集时间，经过转为后@timestamp为日志时间，time字段为采集时间



### 7、filebeat 错误提示

 ERROR   logstash/async.go:256   Failed to publish events caused by: read tcp 192.168.130.191:50706-&gt;192.168.200.25:5054: i/o timeout

总结：

logstash性能问题，修改超时时间减少提示

```
  beats {
    client_inactivity_timeout => 120
    port => 5054

```



