1、记录错误继续执行

```
    for hit in response:
            try:
                print >> f, (hit.logmessage)
        except Exception,e:
                print >> w, Exception,":",e
```

2、读取文件去掉换行

```
line = line.strip('\n')
```

3、readline文件显示去掉换行

```
#python2
print dir_open.readline(), 

#pytheon3
print(line, end = '')
```



