1、查看群集状态

curl 'localhost:9200/\_cat/health?v'

2、查看所有索引

curl 'localhost:9200/\_cat/indices?v'

3、查看blog索引情况

curl -XGET 'localhost:9200/blog?pretty'

4、查看blog索引mapping情况

curl -XGET 'localhost:9200/blog/\_mapping?pretty'

5、查看blog内容，默认返回10个

curl -XGET 'localhost:9200/blog/\_search?pretty'

6、删除索引

curl -XDELETE '192.168.30.4:9200/logstash-service-\*?pretty'

7、关闭索引

curl -XPOST [http://192.168.200.27:9200/logstash-application\*-201902\*/\_close](http://192.168.200.27:9200/logstash-application*-201902*/_close)

8、开启索引

curl -XPOST http://192.168.200.27:9200/logstash-application-web-clean-request\*/\_open

9、查看模板

curl -XGET "[http://127.0.0.1:9200/\_template/logstash?pretty](http://127.0.0.1:9200/_template/logstash?pretty)"

10、创建索引

curl -XPUT 'localhost:9200/logstash-temp-11?pretty'

