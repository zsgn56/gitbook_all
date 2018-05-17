1、python使用elasticsearch\_dsl读取所有数据，输出某个字段内容

```
from elasticsearch import Elasticsearch
from elasticsearch_dsl import Search
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

client = Elasticsearch(['192.168.30.4:9200'])

s = Search(using=client, index="logstash-service-sgbs-2018.05.11")
#    .query("match", loglevel="INFO")   

#s.aggs.bucket('per_tag', 'terms', field='tags') 
f = open("out.txt", "w")
w = open("error.txt", "w")

response = s.scan()
#response = s.execute()
for hit in response:
        try:
                print >> f, (hit.logmessage)
        except Exception,e:
                print >> w, Exception,":",e
f.close()
w.close()

```



