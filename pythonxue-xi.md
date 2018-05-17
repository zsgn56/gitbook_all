1、记录错误继续执行

```
    for hit in response:
            try:
                print >> f, (hit.logmessage)
        except Exception,e:
                print >> w, Exception,":",e

```



