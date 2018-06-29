1、匹配出不以空格、tab开头后为at的行

```
#!/usr/bin/env python
#encoding: utf-8
import re
f=open('sgbs-2018.06.28_out.txt','r')
f1=open('result.txt','w')

for line in f.readlines():
        ret_match= (re.match("[ \t]at ",line) or re.match("java.lang",line));
        if not (ret_match):
                f1.write(line)
f.close()
f1.close
```

2、中文匹配，未验证



```
#!/usr/bin/env python
#encoding: utf-8
import re
f=open('10000.txt','r', encoding='UTF-8')
f1=open('result.txt','w')

for line in f.readlines():
    ss = line.split('|||')
    s = ss[0].strip()
    if((re.search('和 |（\d点|月）',s) or re.match('向|个|谈|和|于|桥|上|省|市|县|聚|乡|镇|\d',s))and len(s) > 4 and (not re.search("上海|镇江",s)) ):
        f1.write(line)
        print(line)    
    # f1.writeline(line+"  !")
f.close()
f1.close()
```



