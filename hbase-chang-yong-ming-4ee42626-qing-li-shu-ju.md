```
/usr/local/hbase/bin/hbase shell
>list 查看表
>count 'AgentInfo'   统计表行数
>scan 'AgentInfo'    扫描表
```

脚本定时清理hbase

参考：[https://blog.csdn.net/babyhuang/article/details/79130277](https://blog.csdn.net/babyhuang/article/details/79130277)

    #!/bin/bash
    echo '--------------程序从这里开始------------'
    basepath=$(cd `dirname $0`; pwd)
    #basepath=$(cd <code>dirname $0</code>; pwd)

    echo '---------------正在创建缓存文件夹--------------'
    firstTime="_$1_$2"
    mkdir $basepath/CacheOfdelete$firstTime
    #touch $basepath/CacheOfdelete$firstTime/data$firstTime.txt
    touch $basepath/CacheOfdelete$firstTime/record$firstTime.txt
    touch $basepath/CacheOfdelete$firstTime/delete$firstTime.sh

    #current1="2018-07-17 16:25:18"
    #current2="2018-07-17 16:29:50"
    current1="$1 $2"
    current2="$3 $4"

    tablename="$5"

    echo 开始时间:$current1
    echo 结束时间:$current2
    startSec=`date -d "$current1" +%s`
    endSec=`date -d "$current2" +%s`

    startTimestamp=$((startSec*1000+`date "+%N"`/1000000))
    endTimestamp=$((endSec*1000+`date "+%N"`/1000000))

    echo $tablename
    echo $startTimestamp
    echo $endTimestamp
    #echo $startTimestamp > $basepath/CacheOfdelete$firstTime/data$firstTime.txt
    ##echo $endTimestamp >> $basepath/CacheOfdelete$firstTime/data$firstTime.txt

    # #######第一步：通过时间戳找到要删除的数据
    # 注：这里只有rowkey和其中一列，因为目的是找到rowkey 
    #echo "scan '$tablename',{ COLUMNS => '$6',TIMERANGE => [$startTimestamp,$endTimestamp]}" | /data/hbase-1.2.6/bin/hbase  shell > $basepath/CacheOfdelete$firstTime/record$firstTime.txt
    echo "scan '$tablename',{ TIMERANGE => [$startTimestamp,$endTimestamp]}" | /usr/local/hbase/bin/hbase  shell > $basepath/CacheOfdelete$firstTime/record$firstTime.txt
    # ######第二步：构建删除数据的shell
    #echo "#!/bin/bash " >> $basepath/CacheOfdelete$firstTime/aa.sh
    echo "#!/bin/bash " >> $basepath/CacheOfdelete$firstTime/delete$firstTime.sh
    echo "exec hbase shell <<EOF " >> $basepath/CacheOfdelete$firstTime/delete$firstTime.sh

    cat $basepath/CacheOfdelete$firstTime/record$firstTime.txt|awk '{print "deleteall '\'$tablename\''", ",", "'\''"$1"'\''"}' tName="$tablename" >> $basepath/CacheOfdelete$firstTime/delete$firstTime.sh

    echo "EOF " >> $basepath/CacheOfdelete$firstTime/delete$firstTime.sh

    # ########第三步：执行删除shell
    #sh $basepath/CacheOfdelete$firstTime/delete$firstTime.sh
    echo '---------------正在删除缓存文件夹--------------'
    #rm -rf $basepath/CacheOfdelete$firstTime
    echo '--------------程序到这里结束------------'


    运行：bash 33.sh  2018-12-01 17:11:52 2018-12-02 17:11:53 TraceV2

pinpoint相关处理

参考：[https://blog.csdn.net/qq\_31331391/article/details/79104831](https://blog.csdn.net/qq_31331391/article/details/79104831)

    echo "scan 'TraceV2', {TIMERANGE => [$yescurrentTimeStamp, $currentTimeStamp]}" |  $binhbase/hbase shell > ./hbase.txt
    if [ `wc -l ./hbase.txt | awk '{print $1}'` -gt 9 ]; then
            sed -i '1,6d;N;$d;P;D' ./hbase.txt
            sleep 2
    cat hbase.txt | awk -F" column" '{s=(gensub(/[[:punct:] ]/, "\\\\&", "g", substr($1,2))); gsub(/\\\\x/,"\\x",s);gsub(/\\`/,"`",s)} {print "deleteall '\''TraceV2'\''", ",", "\""s"\""}' > ./del
            sleep 5
            rm -f $path/hbase.txt
    else
            echo "no data delete"
            exit
    fi



