# JDK1.8之metaspace
jdk1.8种取消了原来的永久代PermGen


![java新老对内存划分对比图解](http://s9.51cto.com/wyfs02/M02/24/3F/wKiom1NNx2OBMyxuAACoTamqmjg470.jpg)

- 之前不管是不是需要，JVM都会吃掉那块空间……如果设置得太小，JVM会死掉；如果设置得太大，这块内存就被JVM浪费了。理论上说，现在你完全可以不关注这个，因为JVM会在运行时自动调校为“合适的大小”；

- 提高Full GC的性能，在Full GC期间，Metadata到Metadata pointers之间不需要扫描了，别小看这几纳秒时间；

- 隐患就是如果程序存在内存泄露，像OOMTest那样，不停的扩展metaspace的空间，会导致机器的内存不足，所以还是要有必要的调试和监控