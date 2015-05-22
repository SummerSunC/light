# JVM监控与调优（hotSpot）
## java启动参数简介
[JVM启动参数大全](http://www.blogjava.net/midstr/archive/2008/09/21/230265.html)
* 标准参数(-），所有的JVM实现都必须实现这些参数的功能，而且向后兼容；
* 非标准参数(-X)，默认jvm实现这些参数的功能，但是并不保证所有jvm实现都满足，且不保证向后兼容；
* 非Stable参数 [非静态参数]（-XX），此类参数各个jvm实现会有所不同，将来可能会随时取消，需要慎重使用；

## 常用java启动参数
### 标准参数
标准参数就是运行java命令时后面加上的参数
> e.g: java -version , java -jar  ……

#### -client 
 设置jvm使用client模式，特点是启动速度比较快，但运行时性能和内存管理效率不高，通常用于客户端应用程序或者PC应用开发和调试。

#### -server
 设置jvm使server模式，特点是启动速度比较慢，但运行时性能和内存管理效率很高，适用于生产环境。在具有64位能力的jdk环境下将默认启用该模式，而忽略-client参数。

#### -DpropertyName=value
定义系统的全局属性值(e.g : -Dspring.profiles.active="dev")。
- 如配置文件地址等，如果value有空格，可以用-Dname="space string"这样的形式来定义，

- 用System.getProperty("propertyName")可以获得这些定义的属性值，在代码中也可以用System.setProperty("propertyName","value")的形式来定义属性

#### -verbose 

这是查询GC问题最常用的命令之一，具体参数如：

* -verbose:class ：
 输出jvm载入类的相关信息，当jvm报告说找不到类或者类冲突时可此进行诊断。
* -verbose:gc ：
 输出每次GC的相关情况，后面会有更详细的介绍。
* -verbose:jni ：
 输出native方法调用的相关情况，一般用于诊断jni调用错误信息。

### 非标准参数

- 输入 java -X 命令可以查看所有的  非标准参数。注释神马的也写的很清楚
- 堆的大小=年轻代大小+年老代大小，堆的大小不包含持久代大小，如果增大了年轻代，年老代相应就会减小，官方默认的配置为年老代大小/年轻代大小=2/1左右（使用-XX:NewRatio可以设置-XX:NewRatio=5，表示年老代/年轻代=5/1）；

![](http://images.cnitblog.com/blog/406312/201312/31173615-f034059f20564bdebdb71e10a3e39d09.png)
> ps Eden 译为伊甸园，一切开始的地方



#### -Xmn 
新生代内存大小的最大值.包含E区和2个S区的总和；（记忆方法：Memory New）
- 使用方法：-Xmn65535，-Xmn1024k，-Xmn512m，-Xmn1g (-Xms,-Xmx也是种写法)

#### -Xms
初始堆的大小，堆的最小值。 （记忆方法：Memory Small）
- 默认值是总共的物理内存/64（且小于1G）
- 默认情况下，当堆中可用内存小于40%(这个值可以用-XX: MinHeapFreeRatio 调整，如-X:MinHeapFreeRatio=30)时，堆内存会开始增加，一直增加到-Xmx的大小；

#### -Xmx 
堆的最大值。（记忆方法：Memory maX）
- 默认值是总共的物理内存/64（且小于1G），如果Xms和Xmx都不设置，则两者大小会相同
- 默认情况下，当堆中可用内存大于70%（这个值可以用-XX: MaxHeapFreeRatio 调整，如-X:MaxHeapFreeRatio=60）时，堆内存会开始减少，一直减小到-Xms的大小；

#### -Xss
设置每个线程的栈内存。
- 默认1M，一般不需要改

#### 非Stable参数（非静态参数）
JVM（Hotspot）中主要的参数可以大致分为3类
- 性能参数（ Performance Options）：用于JVM的性能调优和内存分配控制，如初始化内存大小的设置；
- 行为参数（Behavioral Options）：用于改变JVM的基础行为，如GC的方式和算法的选择；
- 调试参数（Debugging Options）：用于监控、打印、输出等jvm参数，用于显示jvm更加详细的信息；

对于非Stable参数，使用方法有4种：
```
-XX:+<option> 启用选项
-XX:-<option> 不启用选项
-XX:<option>=<number> 给选项设置一个数字类型值，可跟单位，例如 32k, 1024m, 2g
-XX:<option>=<string> 给选项设置一个字符串值，例如-XX:HeapDumpPath=./dump.core
```

- 常见性能参数

| 参数及其默认值 | 描述 |
|--|--|
| -XX:NewSize=2.125m|新生代对象生成时占用内存的默认值|
|-XX:MaxNewSize=size | 新生成对象能占用内存的最大值|
|-XX:MaxPermSize=64m|方法区所能占用的最大内存（非堆内存）|
|-XX:PermSize=64m|方法区分配的初始内存|、
|-XX:SurvivorRatio=8|Eden区域Survivor区的容量比值，如默认值为8，代表Eden：Survivor1：Survivor2=8:1:1|
|-XX:MaxTenuringThreshold=15|对象在新生代存活区切换的次数（坚持过MinorGC的次数，每坚持过一次，该值就增加1），大于该值会进入老年代|
|……|……|

- 常见行为参数

| 参数及其默认值 |描述 |
|--|--|
|-XX:-UseSerialGC|启用串行GC，即采用Serial+Serial Old模式|
|-XX:-UseParallelGC|启用并行GC，即采用Parallel Scavenge+Serial Old收集器组合（-Server模式下的默认组合）|
|-XX:+UseParNewGC|使用ParNew+Serial Old收集器组合|
|-XX:+UseParallelOldGC|使用Parallel Scavenge +Parallel Old组合收集器|
|-XX:+UseConcMarkSweepGC|使用ParNew+CMS+Serial Old组合并发收集，优先使用ParNew+CMS，当用户线程内存不足时，采用备用方案Serial Old收集。|
|……|……|

- 常见调试参数

| 参数及其默认值 |描述 |
|--|--|
|-XX:HeapDumpPath=./java_pid< pid >.hprof|指定导出堆信息时的路径或文件名|
|-XX:-HeapDumpOnOutOfMemoryError|当首次遭遇OOM时导出此时堆中相关信息|
|-XX:-PrintConcurrentLocks|遇到Ctrl-Break后打印并发锁的相关信息，与jstack -l功能相同|
|……|……|

## 优化
### 内存分配策略建议 （!important）
- 生产中设置 -Xms = -Xmx（**最小堆内存等于最大堆内存**），以防止抖动，大小受操作系统和内存大小限制，如果是32位系统，则一般-Xms设置为1g-2g（假设有4g内存），在64位系统上，没有限制，不过**一般为机器最大内存的一半左右**；
- **-XX:PermSize尽量比-XX:MaxPermSize小** ;-XX:MaxPermSize>= 2 * -XX:PermSize, -XX:PermSize> 64m，一般对于4G内存的机器，-XX:MaxPermSize不会超过256m；
- -Xmn，在生产环境，**一般-Xmn的大小是-Xms的1/2左右**，不要设置的过大或过小，过大导致老年代变小，频繁Full GC，过小导致minor GC频繁。
- **-Xss一般是不需要改的，默认值即可**。
- **-XX:SurvivorRatio一般设置8-10左右，推荐设置为10**，也即：Survivor区的大小是Eden区的1/10，一般来说，普通的Java程序应用，一次minorGC后，至少98%-99%的对象，都会消亡，所以，survivor区设置为Eden区的1/10左右，能使Survivor区容纳下10-20次的minor GC才满，然后再进入老年代，这个与-XX:MaxTenuringThreshold的默认值15次也相匹配的。如果XX:SurvivorRatio设置的太小，会导致本来能通过minor回收掉的对象提前进入老年代，产生不必要的full gc；如果XX:SurvivorRatio设置的太大，会导致Eden区相应的被压缩。
- -XX:MaxTenuringThreshold默认为15，也就是说，经过15次Survivor轮换（即15次minor GC），就进入老年代， 如果设置的小的话，则年轻代对象在survivor中存活的时间减小，提前进入年老代，对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代的存活时间，增加在年轻代即被回收的概率。需要注意的是，设置了 -XX:MaxTenuringThreshold，并不代表着，对象一定在年轻代存活15次才被晋升进入老年代，它只是一个最大值，事实上，存在一个动态计算机制，计算每次晋入老年代的阈值，取阈值和MaxTenuringThreshold中较小的一个为准。

### 监控工具和方法
#### jps
jps命令用于查询正在运行的JVM进程，
```
常用的参数为：
    -q:只输出LVMID，省略主类的名称
    -m:输出虚拟机进程启动时传给主类main()函数的参数
    -l:输出主类的全类名，如果进程执行的是Jar包，输出Jar路径
    -v:输出虚拟机进程启动时JVM参数
命令格式:jps [option] [hostid] 
```	
#### jstat
jstat可以实时显示本地或远程JVM进程中类装载、内存、垃圾收集、JIT编译等数据（如果要显示远程JVM信息，需要远程主机开启RMI支持）。如果在服务启动时没有指定启动参数-verbose:gc，则可以用jstat实时查看gc情况。
```
jstat有如下选项：
   -class:监视类装载、卸载数量、总空间及类装载所耗费的时间
   -gc:监听Java堆状况，包括Eden区、两个Survivor区、老年代、永久代等的容量，以用空间、GC时间合计等信息
   -gccapacity:监视内容与-gc基本相同，但输出主要关注java堆各个区域使用到的最大和最小空间
   -gcutil:监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比
   -gccause:与-gcutil功能一样，但是会额外输出导致上一次GC产生的原因
   -gcnew:监视新生代GC状况
   -gcnewcapacity:监视内同与-gcnew基本相同，输出主要关注使用到的最大和最小空间
   -gcold:监视老年代GC情况
   -gcoldcapacity:监视内同与-gcold基本相同，输出主要关注使用到的最大和最小空间
   -gcpermcapacity:输出永久代使用到最大和最小空间
   -compiler:输出JIT编译器编译过的方法、耗时等信息
   -printcompilation:输出已经被JIT编译的方法
命令格式:jstat [option vmid [interval[s|ms] [count]]]
jstat可以监控远程机器，命令格式中VMID和LVMID特别说明：如果是本地虚拟机进程，VMID和LVMID是一致的，
如果是远程虚拟机进程，那么VMID格式是: [protocol:][//]lvmid[@hostname[:port]/servername]，
如果省略interval和count，则只查询一次
```
各列解释如下：
- S0C：S0区容量（S1区相同，略）
- S0U：S0区已使用
- EC：E区容量
- EU：E区已使用
- OC：老年代容量
- OU：老年代已使用
- PC：Perm容量
- PU：Perm区已使用
- YGC：Young GC（Minor GC）次数
- YGCT：Young GC总耗时
- FGC：Full GC次数
- FGCT：Full GC总耗时
- GCT：GC总耗时

#### jinfo
用于查询当前运行这的JVM属性和参数的值。
```
jinfo可以使用如下选项：
   -flag:显示未被显示指定的参数的系统默认值
   -flag [+|-]name或-flag name=value: 修改部分参数
   -sysprops:打印虚拟机进程的System.getProperties()
 命令格式:jinfo [option] pid 
```

#### jmap
用于显示当前Java堆和永久代的详细信息（如当前使用的收集器，当前的空间使用率等）
```
-dump:生成java堆转储快照
-heap:显示java堆详细信息(只在Linux/Solaris下有效)
-F:当虚拟机进程对-dump选项没有响应时，可使用这个选项强制生成dump快照(只在Linux/Solaris下有效)
-finalizerinfo:显示在F-Queue中等待Finalizer线程执行finalize方法的对象(只在Linux/Solaris下有效)
-histo:显示堆中对象统计信息
-permstat:以ClassLoader为统计口径显示永久代内存状态(只在Linux/Solaris下有效)
 命令格式:jmap [option] vmid
其中前面3个参数最重要，如：
查看对详细信息：sudo jmap -heap 309
生成dump文件： sudo jmap -dump:file=./test.prof 309
部分用户没有权限时，采用admin用户：sudo -u admin -H  jmap -dump:format=b,file=文件名.hprof pid
查看当前堆中对象统计信息：sudo  jmap -histo 309：该命令显示3列，分别为对象数量，对象大小，对象名称，通过该命令可以查看是否内存中有大对象；
有的用户可能没有jmap权限：sudo -u admin -H jmap -histo 309 | less
```
#### jhat
用于分析使用jmap生成的dump文件，是JDK自带的工具，使用方法为： jhat -J -Xmx512m [file]
不过jhat没有mat好用，推荐使用mat（Eclipse插件： http://www.eclipse.org/mat ），mat速度更快，而且是图形界面。
#### jstack
用于生成当前JVM的所有线程快照,线程快照是虚拟机每一条线程正在执行的方法,目的是定位线程出现长时间停顿的原因。
```
   -F:当正常输出的请求不被响应时，强制输出线程堆栈
   -l:除堆栈外，显示关于锁的附加信息
   -m:如果调用到本地方法的话，可以显示C/C++的堆栈
命令格式:jstack [option] vmid
```
#### -verbosegc
-verbosegc是一个比较重要的启动参数，记录每次gc的日志
```
与-verbosegc配合使用的一些常用参数为：
   -XX:+PrintGCDetails，打印GC信息，这是-verbosegc默认开启的选项
   -XX:+PrintGCTimeStamps，打印每次GC的时间戳
   -XX:+PrintHeapAtGC：每次GC时，打印堆信息
   -XX:+PrintGCDateStamps (from JDK 6 update 4) ：打印GC日期，适合于长期运行的服务器
   -Xloggc:/home/admin/logs/gc.log：制定打印信息的记录的日志位置
每条verbosegc打印出的gc日志，都类似于下面的格式：
time [GC [<collector>: <starting occupancy1> -> <ending occupancy1>(total occupancy1), <pause time1> secs] <starting occupancy3> -> <ending occupancy3>(total occupancy3), <pause time3> secs] 
如：
```
名词解释
- time：执行GC的时间，需要添加-XX:+PrintGCDateStamps参数才有；
- collector：minor gc使用的收集器的名字。
- starting occupancy1：GC执行前新生代空间大小。
- ending occupancy1：GC执行后新生代空间大小。
- total occupancy1：新生代总大小
- pause time1：因为执行minor GC，Java应用暂停的时间。
- starting occupancy3：GC执行前堆区域总大小
- ending occupancy3：GC执行后堆区域总大小
- total occupancy3：堆区总大小
- pause time3：Java应用由于执行堆空间GC（包括full GC）而停止的时间。







