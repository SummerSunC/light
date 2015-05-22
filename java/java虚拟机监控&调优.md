# JVM监控与调优（hotSpot）
## java启动参数
* 标准参数(-），所有的JVM实现都必须实现这些参数的功能，而且向后兼容；
* 非标准参数(-X)，默认jvm实现这些参数的功能，但是并不保证所有jvm实现都满足，且不保证向后兼容；
* 非Stable参数（-XX），此类参数各个jvm实现会有所不同，将来可能会随时取消，需要慎重使用；

### 标准参数
标准参数就是运行java命令时后面加上的参数
> e.g: java -version , java -jar  ……

#### -client 
 设置jvm使用client模式，特点是启动速度比较快，但运行时性能和内存管理效率不高，通常用于客户端应用程序或者PC应用开发和调试。

#### -server
 设置jvm使server模式，特点是启动速度比较慢，但运行时性能和内存管理效率很高，适用于生产环境。在具有64位能力的jdk环境下将默认启用该模式，而忽略-client参数。

#### -agentlib:libname[=options] 
用于装载本地lib包；
> 其中libname为本地代理库文件名，默认搜索路径为环境变量PATH中的路径，options为传给本地库启动时的参数，多个参数之间用逗号分隔。

> 在Windows平台上jvm搜索本地库名为libname.dll的文件，在linux上jvm搜索本地库名为lisbname.so的文件，搜索路径环境变量在不同系统上有所不同，比如Solaries上就默认搜索LD_LIBRARY_PATH。
 
> 比如：-agentlib:hprof
  用来获取jvm的运行情况，包括CPU、内存、线程等的运行数据，并可输出到指定文件中；windows中搜索路径为JRE_HOME/bin/hprof.dll。

#### -agentpath:pathname[=options] 
 按全路径装载本地库，不再搜索PATH中的路径；其他功能和agentlib相同；



