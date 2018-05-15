1、读取文本，按照内容重复次数统计

```
import operator
f = open('pom.txt','r')
count_dict = {}
result = list()
for line in open('pom.txt','r'):
    line = f.readline()
    line = line.strip()
    count = count_dict.setdefault(line, 0)
    count+=1
    count_dict[line] = count
sorted_count_dict = sorted(count_dict.iteritems(), key=operator.itemgetter(1), reverse=True)
for item in sorted_count_dict:
     print "%s,%d" % (item[0], item[1])

f.close()
```



