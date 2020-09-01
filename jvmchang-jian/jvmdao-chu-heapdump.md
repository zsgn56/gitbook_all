1、jmap

jmap -heap pid

jmap -dump:format=b,file=temp\_heapdump.hprof pid

2、jcmd

jcmd 51 VM.command\_line &gt;&gt; boot.dump

jcmd 51 VM.system\_properties &gt;&gt;sys.prop.dump

jcmd 51 GC.class\_histogram &gt;&gt; clz.dump

jcmd 51 GC.heap\_dump  filename=Myheapdump

