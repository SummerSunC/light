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
- 
- 输入 java -X 命令可以查看所有的  非标准参数。注释神马的也写的很清楚
- 堆的大小=年轻代大小+年老代大小，堆的大小不包含持久代大小，如果增大了年轻代，年老代相应就会减小，官方默认的配置为年老代大小/年轻代大小=2/1左右（使用-XX:NewRatio可以设置-XX:NewRatio=5，表示年老代/年轻代=5/1）；
建议在开发测试环境可以用Xms和Xmx分别设置最小值最大值，但是**在线上生产环境，Xms和Xmx设置的值必须一样，原因与年轻代一样——防止抖动；**



![](http://images.cnitblog.com/blog/406312/201312/31173615-f034059f20564bdebdb71e10a3e39d09.png)
> ps Eden 译为伊甸园，一切开始的地方



#### -Xmn
新生代内存大小的最大值.包含E区和2个S区的总和；
- 使用方法：-Xmn65535，-Xmn1024k，-Xmn512m，-Xmn1g (-Xms,-Xmx也是种写法)

#### -Xms
初始堆的大小，堆的最小值。
- 默认值是总共的物理内存/64（且小于1G）
- 默认情况下，当堆中可用内存小于40%(这个值可以用-XX: MinHeapFreeRatio 调整，如-X:MinHeapFreeRatio=30)时，堆内存会开始增加，一直增加到-Xmx的大小；

#### -Xmx
堆的最大值。
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
常见性能参数
| 参数及其默认值 | 描述 |
|--|--|
| -XX:NewSize=2.125m|新生代对象生成时占用内存的默认值|
|-XX:MaxNewSize=size | 新生成对象能占用内存的最大值|
|-XX:MaxPermSize=64m|方法区所能占用的最大内存（非堆内存）|
|……|……|

常见行为参数
| 参数及其默认值 |描述 |
|--|--|
|-XX:-UseSerialGC|启用串行GC，即采用Serial+Serial Old模式|
|-XX:-UseParallelGC|启用并行GC，即采用Parallel Scavenge+Serial Old收集器组合（-Server模式下的默认组合）|
|-XX:+UseParNewGC|使用ParNew+Serial Old收集器组合|
|-XX:+UseParallelOldGC|使用Parallel Scavenge +Parallel Old组合收集器|
|-XX:+UseConcMarkSweepGC|使用ParNew+CMS+Serial Old组合并发收集，优先使用ParNew+CMS，当用户线程内存不足时，采用备用方案Serial Old收集。|
|……|……|

常见调试参数
| 参数及其默认值 |描述 |
|--|--|
|-XX:HeapDumpPath=./java_pid< pid >.hprof|指定导出堆信息时的路径或文件名|
|-XX:-HeapDumpOnOutOfMemoryError|当首次遭遇OOM时导出此时堆中相关信息|
|-XX:-PrintConcurrentLocks|遇到Ctrl-Break后打印并发锁的相关信息，与jstack -l功能相同|
|……|……|


	










