



1、早期的例子  pip install elasticsearch

```
from elasticsearch import Elasticsearch
#from elasticsearch_dsl import Search,Q,scan

# by default we connect to localhost:9200
_es = Elasticsearch()


print('Search all...')
_query_all = {
  'query': {
    'match_all': {}
  }
}

_query_logmessage = {
  'query': {
    'match': {
                "logmessage":"zk SyncConnected" }
  }
}


_searched = _es.search(index='logstash-service-ssps-2018.05.13', body=_query_logmessage)
print(_searched)
```

2、用dsl方式 pip install elasticsearch\_dsl

```
'match': {

            "logmessage":"zk SyncConnected" }
```

}

}

\_searched = \_es.search\(index='logstash-service-ssps-2018.05.13', body=\_query\_logmessage\)

print\(\_searched\)

