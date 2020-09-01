[https://blog.csdn.net/wskinght/article/details/17081641](https://blog.csdn.net/wskinght/article/details/17081641)

jvm区域总体分两类，heap区和非heap区。heap区又分：Eden Space（伊甸园）、Survivor Space\(幸存者区\)、Tenured Gen（老年代-养老区）。 非heap区又分：Code Cache\(代码缓存区\)、Perm Gen（永久代）、Jvm Stack\(java虚拟机栈\)、Local Method Statck\(本地方法栈\)

Java堆（heap区）中是JVM管理的最大一块内存空间。主要存放对象实例。

在JAVA中堆被分为两块区域：新生代（young）、老年代（old）。

堆大小=新生代+老年代；（新生代占堆空间的1/3、老年代占堆空间2/3）

新生代又被分为了eden、from survivor、to survivor\(8:1:1\)

**年轻代中的GC**

HotSpot JVM把年轻代分为了三部分：1个Eden区和2个Survivor区（分别叫from和to）。默认比例为8：1,为啥默认会是这个比例，接下来我们会聊到。一般情况下，新创建的对象都会被分配到Eden区\(一些大对象特殊处理\),这些对象经过第一次Minor GC后，如果仍然存活，将会被移到Survivor区。对象在Survivor区中每熬过一次Minor GC，年龄就会增加1岁，当它的年龄增加到一定程度时，就会被移动到年老代中。

因为年轻代中的对象基本都是朝生夕死的\(80%以上\)，所以在年轻代的垃圾回收[算法](http://lib.csdn.net/base/datastructure)使用的是复制算法，复制算法的基本思想就是将内存分为两块，每次只用其中一块，当这一块内存用完，就将还活着的对象复制到另外一块上面。复制算法不会产生内存碎片。

在GC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值\(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置\)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，“From”和“To”会交换他们的角色，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。

![](/assets/gc.png)![](/assets/heap.png)

