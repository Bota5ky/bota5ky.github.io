### 1. 运行时数据区域

<center><img src="jvm.png" style="zoom:50%"></center>

- `虚拟机栈` : Java 虚拟机栈是线程私有的数据区，Java 虚拟机栈的生命周期与线程相同，虚拟机栈也是局部变量的存储位置。方法在执行过程中，会在虚拟机栈种创建一个 `栈帧(stack frame)`
- `本地方法栈`: 本地方法栈也是线程私有的数据区，本地方法栈存储的区域主要是 Java 中使用 `native` 关键字修饰的方法所存储的区域
- `程序计数器`：程序计数器也是线程私有的数据区，这部分区域用于存储线程的指令地址，用于判断线程的分支、循环、跳转、异常、线程切换和恢复等功能，这些都通过程序计数器来完成。
- `方法区`：方法区是各个线程共享的内存区域，它用于存储虚拟机加载的 类信息、常量、静态变量、即时编译器编译后的代码等数据
- `堆`：堆是线程共享的数据区，堆是 JVM 中最大的一块存储区域，所有的对象实例都会分配在堆上
- `运行时常量池`：运行时常量池又被称为 `Runtime Constant Pool`，这块区域是方法区的一部分，它的名字非常有意思，它并不要求常量一定只有在编译期才能产生，也就是并非编译期间将常量放在常量池中，运行期间也可以将新的常量放入常量池中，String 的 intern 方法就是一个典型的例子

### 2. 垃圾回收

可作为gc roots的对象：

- **虚拟机栈**中引用的对象
- 方法区中**静态属性**引用的对象
- 方法区中**常量**引用的对象
- **本地方法栈**中JNI引用的对象

JDK1.2之后对引用进行了扩充：

- 强引用：不会被回收
- 软引用：内存要溢出的时候会回收
- 弱引用：只要发生垃圾回收，就会被回收
- 虚引用：在回收时会收到一个通知

### 3. HotSpot垃圾分代回收算法

默认情况下新生代占1/3，老年代占2/3

绝大多数对象在新生代中被创建，这里的垃圾回收非常频繁且速度很快

新生代通常采用复制算法，由于存活对象少，复制成本很低

新生代分为 Eden、Surivivor  from、Surivivor to区，占比8比1比1

Eden填满后触发一次新生代的垃圾回收，称为minor gc，存活对象复制到任一Surivivor区，然后将Eden区清空即可完成这次gc，Surivivor from区的存活对象会复制到另一个Surivivor to区，这里需要保证to区为空

- 存活超过复制次数阈值（默认15）会被复制到老年代

- Surivivor空间不够容纳存活对象时，也会直接进入老年代
- 大数组或者特别大的字符串

老年代通常使用标记整理算法进行回收，将存活对象向一端进行移动，称为major gc

### 4. Serial

JDK1.3版本之前唯一的串行垃圾回收器，Stop The World

### 5. ParNew

多线程垃圾回收，只负责新生代的垃圾回收，可以配合Serial Old和Concurrent Mark Swap处理老年代

### 6. Parallel Scavenge

也是新生代的收集器，可控制吞吐量，gc自适应，配合Parallel old处理老年代

### 7. G1/CMS并发标记原理

- 三色标记
  - 白色：没有被访问过 -> 垃圾对象
  - 黑色：包括其引用都被访问过
  - 灰色：被访问过，但还存在一些引用没有被访问

**对象消失问题**：扫描过程中插入了一条或多条从黑色对象到白色对象的新引用，并且同时去掉了灰色对象到该白色对象的直接引用或者间接引用。

解决方法，破坏上述两个条件之一即可：

- 增量更新：记录引用关系，并发扫描结束后根据记录重新扫描一次 -> CMS

- 原始快照（SATB）：记录 -> G1

[JVM_垃圾收集之三色标记算法详解](https://blog.csdn.net/chuige2013/article/details/129659171)

CMS缺点

- 占用CPU资源，不超过25%
- 浮动垃圾
- 内存碎片