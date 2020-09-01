### 测试结论：

1、主要瓶颈在logstash本身的grok处理能力上，次要在cpu，IO的瓶颈远远未达到

2、输出上：json的性能比dots的性能高1倍以上

3、单台logstash日志处理容量上限：10G/小时

正式环境上：Events Received Rate 一般最高150+/s 偶尔峰值250/s，630k/小时，150m/小时，Event Latency 正常在10ms左右

#### 测试配置

pipeline.batch.size: 125

cpu：2核G

jvm 内存 ：4G

#### 基础测试：

```
导入1000w条数据，容量大概1G ，耗时3分钟， 最高速率[41.5kiB/s]
```

input\_test.conf

```
input {
    generator {
        count => 10000000
        message => '{"key1":"value1","key2":[1,2],"key3":{"subkey1":"subvalue1"}}'
        codec => json
    }
}

output {

    stdout {
        codec => dots
    }

  kafka {
    bootstrap_servers => "192.168.200.24:9092,192.168.200.25:9092"
    topic_id => "test1"
  }
}
```

./bin/logstash -f input\_test.conf \| pv -abt &gt; /dev/null

```
导出1300w条数据，总容量4.85GiB，耗时35分钟， 最高[20.67MiB/s]
```

output\_test.conf

```
input {
  kafka {
        bootstrap_servers => "192.168.200.24:9092,192.168.200.25:9092"
        topics => ["test1"]
        codec => "json"
        add_field => {"app_domain" =>"cloud"}
  }
}

output {
    stdout {
      codec => dots
    }
}
```

./bin/logstash -f output\_test.conf \| pv -abt &gt; /dev/null

#### 模拟日志测试：

不使用正则规则

```
导入1000w条数据，数据容量大概2.3G，耗时4分钟， 最高速率[37.5kiB/s]
cpu使用在60-80%之间
```

使用正则规则

```
导入1000w条数据，数据容量大概2.3G，耗时15分钟， 最高速率[10.5kiB/s]

cpu使用在85%以上，瓶颈在cpu上
```

input\_test.conf

```
input {
    generator {
        count => 10000000
        message => '2019-02-12 00:02:09.510 INFO  [DubboServerHandler-172.18.35.8:20010-thread-400] [54763db1-4354-4be4-82d7-eabf25e73ddb] h.s.s.o.r.RedEnvelopeCodeOpenServiceImpl - javamatchBestRedEnvelopeCode zabisf sfli 901-123llll id XXX91-14-1-X1l1jl'
#    codec => plain
    }
}

filter {


        # 对于服务的匹配
        grok {
            patterns_dir => "/etc/logstash/conf.d/patterns/"
            match => [
              "message", "%{OUTPUT_MAIN_LOG}"
             ]
            remove_field => "message"
          }

    # 新增字段，记录收集时间
    ruby {
      code => "event.set('receivetime', event.get('@timestamp').time.localtime )"
    }


    # 时间戳转换，并删除原始的时间字段
    date {
      match => [ "timestamp",  "yyyy-MM-dd HH:mm:ss.SSS","yyyy-MM-dd HH:mm:ss,SSS", "yyyy-MM-dd-HH-mm-ss,SSS", "yyyy-MM-dd-HH-mm-ss-SSS" ]
      locale => "en"
      # timezone => "+08:00"
      remove_field => "timestamp"
    }


    # 新增字段，将索引创建时间改为每天零时
    ruby {
      code => "event.set('IndexDay', event.get('@timestamp').time.localtime.strftime('%Y%m%d'))"
    }

}


output {

    stdout {
        codec => dots
    }

  kafka {
    bootstrap_servers => "192.168.200.24:9092,192.168.200.25:9092"
    topic_id => "test1"
    codec => "json"
  }
}
```

./bin/logstash -f input\_test.conf \| pv -abt &gt; /dev/null

```
导出1100w条数据，总容量4.58GiB，耗时11分钟， 最高[10MiB/s]
```

output\_test.conf

```
input {
  kafka {
        bootstrap_servers => "192.168.200.24:9092,192.168.200.25:9092"
        topics => ["test1"]
        codec => "json"
        add_field => {"app_domain" =>"cloud"}
  }
}

output {
    stdout {
        codec => json
#      codec => dots
    }
}
```

./bin/logstash -f output\_test.conf \| pv -abt &gt; /dev/null

#### 测试配置

pipeline.batch.size: 500 （可能配置没生效，后续还要再确认）

cpu：2核

jvm 内存 ：8G

#### 模拟日志测试：

结果和没调整配置前相同

```
导入1000w条数据，数据容量大概2.3G，耗时15分钟， 最高速率[10.5kiB/s]

cpu使用在85%以上，瓶颈在cpu上
```



