注：template模板中名称和logstash定义的索引相同，即可识别

1、通过kibana的devtools创建自定义template

```
PUT _template/template_logstash-application
{
  "order": 2, #数字高模板优先级越高
  "template": "logstash-application*", #index_patterns替代template
  "settings": {
    "index": {
      "number_of_shards": 3,
      "number_of_replicas": 0,
      "refresh_interval": "30s"
    }
  }
}
```

2、通过kibana的devtools查看模板正常

```
GET _template/template_logstash-application
```

3 命令行确认插入数据正常

```
curl -XGET 'localhost:9200/logstash-application-backend-backend-ccs-main-20190508?pretty'
```



