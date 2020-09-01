--verbose --debug 启动命令加调试信息

#### 1、**pipeline-workers**

运行 filter 和 output 的 pipeline 线程数量。默认是 CPU 核数

#### 2、**pipeline-batch-size**

.每个 Logstash pipeline 线程，在执行具体的 filter 和 output 函数之前，最多能累积的日志条数。默认是 125 条。越大性能越好，同样也会消耗越多的 JVM 内存

vim /usr/local/logstash/config/logstash.yml

pipeline.batch.size: 250

