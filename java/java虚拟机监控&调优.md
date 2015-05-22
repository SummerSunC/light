# JVM监控与调优（hotSpot）
## java启动参数简介
[JVM启动参数大全](http://www.blogjava.net/midstr/archive/2008/09/21/230265.html)
* 标准参数(-），所有的JVM实现都必须实现这些参数的功能，而且向后兼容；
* 非标准参数(-X)，默认jvm实现这些参数的功能，但是并不保证所有jvm实现都满足，且不保证向后兼容；
* 非Stable参数（-XX），此类参数各个jvm实现会有所不同，将来可能会随时取消，需要慎重使用；

## 常用java启动参数
### 标准参数
标准参数就是运行java命令时后面加上的参数
> e.g: java -version , java -jar  ……

#### -client 
 设置jvm使用client模式，特点是启动速度比较快，但运行时性能和内存管理效率不高，通常用于客户端应用程序或者PC应用开发和调试。

#### -server
 设置jvm使server模式，特点是启动速度比较慢，但运行时性能和内存管理效率很高，适用于生产环境。在具有64位能力的jdk环境下将默认启用该模式，而忽略-client参数。

#### -DpropertyName=value
定义系统的全局属性值，如配置文件地址等，如果value有空格，可以用-Dname="space string"这样的形式来定义，用System.getProperty("propertyName")可以获得这些定义的属性值，在代码中也可以用System.setProperty("propertyName","value")的形式来定义属性

#### -verbose 

这是查询GC问题最常用的命令之一，具体参数如：

* -verbose:class ：
 输出jvm载入类的相关信息，当jvm报告说找不到类或者类冲突时可此进行诊断。
* -verbose:gc ：
 输出每次GC的相关情况，后面会有更详细的介绍。
* -verbose:jni ：
 输出native方法调用的相关情况，一般用于诊断jni调用错误信息。

### 非标准参数






