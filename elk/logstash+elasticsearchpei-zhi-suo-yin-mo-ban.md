在使用logstash收集日志的时候，我们一般会使用logstash自带的动态索引模板，虽然无须我们做任何定制操作，就能把我们的日志数据推送到elasticsearch索引集群中，但是在我们查询的时候，就会发现，默认的索引模板常常把我们不需要分词的字段，给分词了，这样以来，我们的比较重要的聚合统计就不准确了： 

如果使用的是logstash的默认模板，它会按-切分机器名，这样以来想统计那台机器上的收集日志最多就有问题了，所以这时候，就需要我们自定义一些索引模板了：   
  
在logstash与elasticsearch集成的时候，总共有如下几种使用模板的方式：   
  
（1）使用默认自带的索引模板 ，大部分的字段都会分词，适合开发和时候快速验证使用   
（2）在logstash收集端自定义配置模板，因为分散在收集机器上，维护比较麻烦   
（3）在elasticsearc服务端自定义配置模板，由elasticsearch负责加载模板，可动态更改，全局生效，维护比较容易   
  
以上几种方式：   
  
使用第一种，最简单，无须任何配置   
使用第二种，适合小规模集群的日志收集，需要在logstash的output插件中使用template指定本机器上的一个模板json路径， 例如  template =&gt; "/tmp/logstash.json" 

使用第三种，适合大规模集群的日志收集，如何配置，主要配置logstash的output插件中两个参数：

```
manage_template =>false //关闭logstash自动管理模板功能  

template_name =>"crawl" //映射模板的名字  
```

如果使用了，第三种需要在elasticsearch的集群中的config/templates路径下配置模板json，在elasticsearch中索引模板可分为两种： 



### **1、静态模板**

 适合索引字段数据固定的场景，一旦配置完成，不能向里面加入多余的字段，否则会报错   
优点：scheam已知，业务场景明确，不容易出现因字段随便映射从而造成元数据撑爆es内存，从而导致es集群全部宕机   
缺点：字段数多的情况下配置稍繁琐   
  
一个静态索引模板配置例子如下： 

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

```
{  
  "crawl" : {  
      "template": "crawl-*",  
        "settings": {  
            "index.number_of_shards": 3,  
            "number_of_replicas": 0   
        },  
    "mappings" : {  
      "logs" : {  
        "properties" : {  
          "@timestamp" : {  
            "type" : "date",  
            "format" : "dateOptionalTime",  
            "doc_values" : true  
          },  
          "@version" : {  
            "type" : "string",  
            "index" : "not_analyzed",  
        "doc_values" : true      
          },  
          "cid" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "crow" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "erow" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "host" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "httpcode" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "message" : {  
            "type" : "string"  
          },  
          "path" : {  
            "type" : "string"  
          },  
          "pcode" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "pro" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "ptype" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "save" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "t1" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "t2" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "t3" : {  
            "type" : "string",  
            "index" : "not_analyzed"  
          },  
          "url" : {  
            "type" : "string"  
          }  
        }  
      }  
    }  
  }  
}  
```

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)





### **2、动态模板 **

适合字段数不明确，大量字段的配置类型相同的场景，多加字段不会报错   
  
优点：可动态添加任意字段，无须改动scheaml，   
缺点：如果添加的字段非常多，有可能造成es集群宕机   
  
如下的一个logstash的动态索引模板，只设置message字段分词，其他的字段默认不分词

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

```
{  
  "template" : "crawl-*",  
  "settings" : {  
   "index.number_of_shards": 5,  
   "number_of_replicas": 0    
  
},  
  "mappings" : {  
    "_default_" : {  
      "_all" : {"enabled" : true, "omit_norms" : true},  
      "dynamic_templates" : [ {  
        "message_field" : {  
          "match" : "message",  
          "match_mapping_type" : "string",  
          "mapping" : {  
            "type" : "string", "index" : "analyzed", "omit_norms" : true,  
            "fielddata" : { "format" : "disabled" }  
          }  
        }  
      }, {  
        "string_fields" : {  
          "match" : "*",  
          "match_mapping_type" : "string",  
          "mapping" : {  
            "type" : "string", "index" : "not_analyzed", "doc_values" : true  
          }  
        }  
      } ],  
      "properties" : {  
        "@timestamp": { "type": "date" },  
        "@version": { "type": "string", "index": "not_analyzed" },  
        "geoip"  : {  
          "dynamic": true,  
          "properties" : {  
            "ip": { "type": "ip" },  
            "location" : { "type" : "geo_point" },  
            "latitude" : { "type" : "float" },  
            "longitude" : { "type" : "float" }  
          }  
        }  
      }  
    }  
  }  
}
```

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

