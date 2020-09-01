### 基本原则

-Xmx -Xms一致，避免GC

jvm分配内存为物理内存的1/2

内存最高不超过32G

官方建议：old pool   [https://www.elastic.co/blog/found-sizing-elasticsearch](https://www.elastic.co/blog/found-sizing-elasticsearch)

* Below 75%: No worries.
* 75% to 85%: Acceptable if performance is satisfactory and load is expected to be stable.
* Above 85%: Now is the time for action. Either reduce memory consumption or add more memory.

### 查看内存使用情况

系统中java工具：jmap   -heap PID

concurrent mark-sweep generation 采用CMS的老年代



