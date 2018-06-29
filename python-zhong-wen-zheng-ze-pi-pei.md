1、

```
#!/usr/bin/env python
#encoding: utf-8
import re
f=open('10000.txt','r', encoding='UTF-8')
f1=open('result.txt','w')

for line in f.readlines():
    ss = line.split('|||')
    s = ss[0].strip()
    if((re.search('和 |（\d点|月）',s) or re.match('向|镇|\d',s))and len(s) > 4 and (not re.search("上海|镇江",s)) ):
        f1.write(line)
        print(line)    
    # f1.writeline(line+"  !")
f.close()
f1.close()
```



