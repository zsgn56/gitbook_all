top   定位占cpu

pidpidstat -p pid  -u 1 3 -t   定位占cpu线程

jstack pid  查看java堆栈信息

现在将线程的PID转换成16进制。

```
 19558转换成16进制的话，是4c66.

 我们在上面jstack中打印出来的dump信息里面，搜索4c66。可以占资源的方法
```



