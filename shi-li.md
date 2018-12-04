1、读取elk，统计错误日志数量，输出明细

```
import datetime

yes_time = datetime.datetime.now()-datetime.timedelta(days=1)

yes_day  = '%d%s%d%d%s%d' % (yes_time.year,'.',0,yes_time.month,'.',yes_time.day)
#print "yes_time.year" + "yes_time.month"


o = open(yes_day+"_count.txt", "w")
o1 = open(yes_day+"_all.txt", "w")
o.close()
o1.close()
file = open("project")
for line in file:
        line = line.strip()
        filename = line+'-'+yes_day
        o = open(yes_day+"_count.txt", "a")
        print >> o, line + ": ",
        from elasticsearch import Elasticsearch
        from elasticsearch_dsl import Search
        import sys
        reload(sys)
        sys.setdefaultencoding('utf-8')
        client = Elasticsearch(['192.168.30.4:9200'])

        s = Search(using=client, index="logstash-"+filename).query("match", loglevel="ERROR")
    #.query("match", loglevel="ERROR")   
#s.aggs.bucket('per_tag', 'terms', field='tags') 
        f = open(filename+"_out.txt", "w")
        w = open(filename+"_error.txt", "w")
        o = open(yes_day+"_count.txt", "a")
        count = 0
        response = s.scan()
        #response = s.execute()
        for hit in response:
                count = count + 1
                try:
        #                        print >> f, (hit.time)
                         print >> f, (hit.logmessage)
                except Exception,e:
                         print >> w, Exception,":",e
        print >> o, count
        f.close()
        w.close()
        o.close()

        import re
        f=open(filename+"_out.txt",'r')
        f1=open('result_tmp.txt','w')
        for line in f.readlines():
                ret_match= (re.match("[ \t]at ",line) or re.match("java.lang",line));
                if not (ret_match):
                        f1.write(line)
        f.close()
        f1.close()


        import operator
        lf = open('result_tmp.txt','r')
        lf2 = open(yes_day+"_all.txt",'a')
        print >> lf2, "total:" + filename
        count_dict = {}
        result = list()
        for line in lf:
                line = line.strip()
                count = count_dict.setdefault(line, 0)
                count+=1
                count_dict[line] = count
        sorted_count_dict = sorted(count_dict.iteritems(), key=operator.itemgetter(1), reverse=True)
        for item in sorted_count_dict:
                 print>> lf2,  "%d : %s" % (item[1], item[0])

        lf.close()
        lf2.close()
```

存在问题：无法判断素引是否存在，导致报错后续无法进行，目前只能通过project中顺序进行控制

