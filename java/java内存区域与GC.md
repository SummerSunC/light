# java 内存区域与GC机制

### 写在前面
> 默认虚拟机：HotSpot

## java内存区域
### 内存区域
- 程序计数器
- 栈
    - 虚拟机栈
    - 本地方法栈
- 堆
    - 普通堆区
    - 方法区（版本、field、方法、接口等信息）、final常量、静态变量、编译器即时编译的代码等，基本不会进行垃圾回收，也不是绝对）
        - 运行时常量池（存储编译时即时生成的字面常量、符号引用、翻译出来的直接引用）

    
### java对象访问方式

``` java
Object obj = new Object()
```
这里访问的方式为：
- Object obj表示一个本地引用，存储在JVM栈的本地变量表中，表示一个reference类型数据
- new Object()作为实例对象数据存储在堆中
- 堆中还记录了Object类的类型信息（接口、方法、field、对象类型等）的地址，这些地址所执行的数据存储在方法区中

### java内存分配方式
这里的的内存分配实际上是在堆上的内存分配。java采取的内存分配机智是：**分代分配，分代回收**。
> 年轻代（Young Generation）
> 年老代（Old Generation）
> 永久代（Permanent Generation，在HotSpot虚拟机中是方法区）