### 1. CMS (Concurrent-Mark-Sweep)

在 JDK 1.5 时期，HotSpot 推出了一款在强交互应用中几乎可认为有划时代意义的垃圾收集器：CMS 收集器，这款收集器是 HotSpot 虚拟机中第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程同时工作。

CMS 收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间。停顿时间越短（低延迟）就越适合与用户交互的程序，良好的响应速度能提开用户体验。

- 目前很大一部分的 Java 应用集中在互联网站或者 B/S 系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。CMS 收集器就非常符合这类应用的需求。

CMS 的垃圾收集算法采用标记-清除算法，并且也会“stop-the-world”

不幸的是，CMS 作为老年代的收集器，却无法与 JDK 1.4. 中已经存在的新生代收集器 Parallel Scavenge 配合工作，所以在 JDK 1.5 中使用 CMS 来收集老年代的时候，新生代只能选择 ParNew 或者 Serial 收集器中的一个。

在 G1 出现之前，CMS 使用还是非常广泛的。一直到今天，仍然有很多系统使用 CMS GC。

**CMS 垃圾清理过程：**当堆内存使用率达到某一阈值时，便开始进行回收。

- 初始标记（STW）：暂时时间非常短，标记与 GC Roots 直接关联的对象。
- 并发标记（最耗时）：从 GC Roots 开始历整个对象图的过程，不会停顿用户线程。
- 重新标记（STW）：修复并发标记环节，因为用户线程的执行，导致数据的不一致性问题
- 并发清理(最耗时)

有人会觉得既然 Mark Sweep 会造成内存碎片，那么为什么不把算法换成 Mark Compact 呢？

因为当并发清除的时候，用 Compact 整理内存的话，原来的用户线程使用的内存还怎么用呢？要保证用户线程能继续执行，前提的它运行的资源不受影响嘛。Mark-Compact 更适合“stop the World”这种场景下使用。

参数：

- `-XX:+UseConcMarkSweepGC`手动指定使用 CMS 收集器执行内存回收任务
  - 开启该参数后会自动将-XX:+UseParNewGC打开。即：ParNew（Young区用）+CMS（old区用）+Serial old 的组合。
- `-XX:CMSInitiatingOccupanyFraction`设置堆内存使用率的阙值，一旦达到该阙值，便开始进行回收。
    - JDK5 及以前版本的默认值为 68,即当老年代的空间使用率达到 68% 时，会执行一次 CMS 回收。JDK6 及以上版本默认值为 92%
    - 如果内存增长缓慢，则可以设置一个稍大的值，大的阙值可以有效降低CMS的触发频率，减少老年代回收的次数可以较为明显地改善应用程序性能。反之，如果应用程序内存使用率增长很快，则应该降低这个阙值，以避免频繁触发老年代串行收集器。因此通过该选项便可以有效降低Full GC 的执行次数。
- `-XX:+UseCMSCompactAtFulICollection`用于指定在执行完 Full GC 后对内存空间进行压缩整理，以此避免内存碎片的产生。不过由于内存压缩整理过程无法并发执行，所带来的问题就是停顿时间变得更长了。
- `-XX:CMSFulGCsBeforeCompaction`设置在执行多少次 Full GC 后对内存空间进行压缩整理。

### 2. G1 (Garbage First)

G1 将内存划分为一个个的 region。内存的回收是以 region 作为基本单位的。Region 之间是复制算法，但整体上实际可看作是标记-压缩（Mark-Compact）算法，两种算法都可以避免内存碎片。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次 GC。尤其是当 Java 堆非常大的时候，G1 的优势更加明显。

参数：

- `-XX: +UseG1GC`手动指定使用G1收集器执行内存回收任务。
- `-XX:G1HeapRegionSize`设置每个 Region 的大小。值是 2 的幂，范围是 1MB 到 32MB 之间，目标是根据最小的 Java 堆大小划分出约 2048 个区域。默认是堆内存的 1/2000。
- `-XX:MaxGCPauseMillis` 设置期望达到的最大 GC 停顿时间指标（JVM会尽力实现，但不保证达到）。默认值是200ms。（主要优化参数）
- `-XX:ParallelGCThread`设置 STW 时 GC 线程数的值。最多设置为 8。
- `-XX:ConcGCThreads`设置并发标记的线程数。将n设置为并行垃圾回收线程数（ParallelGCThreads）的1/4左右。
- `-XX:InitiatingHeapOccupancyPercent`设置触发并发 GC 周期的 Java 堆占用率阙值。超过此值，就触发GC。默认值是45。

G1 GC 的垃圾回收过程主要包括如下三个环节：

- 年轻代 GC（Young GC）
- 老年代并发标记过程（Concurrent Marking）
- 混合回收 (Mixed GC)：整个 Young Region 加一部分 Old Region
- （如果需要，单线程、独占式、高强度的 Full GC 还是继续存在的。它针对 GC 的评估失败提供了一种失败保护机制，即强力回收。）

### 3. GC 评估指标

- **吞吐量：**程序的运行时间/(程序的运行时间+内存回收的时间)
- 垃圾收集开销：吞吐量的补数，垃圾收集器所占时间与总时间的比例。
- **暂停时间：** 执行垃圾收集时，程序的工作线程被暂停的时间。
- 收集频率：相对于应用程序的执行，收集操作发生的频率。
- 内存占用：Java 堆区所占的内存大小。
- 快速：一个对象从诞生到被回收所经历的时间。

吞吐量优先：单位时间内，STW 的时间最短
响应时间优先：尽可能让单次 STW 的时间最短

现在 JVM 调优标准：在最大吞吐量优先的情况下，降低停顿时间。

### 4. 各 GC 使用场景

|  垃圾收集器  |   分类   |   作用位置   |   使用算法   |     特点     |               适用场景               |
| :----------: | :------: | :----------: | :----------: | :----------: | :----------------------------------: |
|    Serial    |   串行   |    新生代    |   复制算法   | 响应速度优先 |    适用于单CPU环境下的client模式     |
|    ParNew    |   并行   |    新生代    |   复制算法   | 响应速度优先 |  多CPU环境Server模式下与CMS配合使用  |
|   Parallel   |   并行   |    新生代    |   复制算法   |  吞吐量优先  | 适用于后台运算而不需要太多交互的场景 |
|  Serial Old  |   串行   |    老年代    |   标记压缩   | 响应速度优先 |    适用于单CPU环境下的client模式     |
| Parallel Old |   并行   |    老年代    |   标记压缩   |  吞吐量优先  | 适用于后台运算而不需要太多交互的场景 |
|     CMS      |   并发   |    老年代    |   标记清除   | 响应速度优先 |        适用于互联网或B/S业务         |
|      G1      | 并发并行 | 新生代老年代 | 标记压缩复制 | 响应速度优先 |            面向服务端应用            |

### 5. JVM 常用参数

|                           options                            |                           参数含义                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                         -verbose:gc                          |              输出GC日志信息，默认输出到标准输出              |
|                         -XX:+PrintGC                         |                输出GC日志。类似：-verbose:gc                 |
|                     -XX:+PrintGCDetails                      | 在发生垃圾回收时打印内存回收详细的日志并在进程退出时输出当前内存各区域分配情况 |
|                    -XX:+PrintGCTimeStamps                    |                     输出GC发生时的时间截                     |
|                    -XX:+PrintGCDateStamps                    | 输出GC发生时的时间戳（以日期的形式，如2013-05-04T21:53:59.234+0800) |
|                      -XX:+PrintHeapAtGC                      |                每一次GC前和GC后，都打印堆信息                |
| -Xloggc:<file><br />-XX:+UseGCLogFileRotation<br />-XX:NumberOfGCLogFiles=5 |    表示把GC日志写入到一个文件中去，而不是打印到标准输出中    |
|                           -Xss512K                           |                     设置每个线程的栈大小                     |
|     -XX:MetaspaceSize=64m<br />-XX:MaxMetaspaceSize=60m      |                        设置元空间大小                        |
| -XX:+HeapDumpOnOutOfMemoryError<br />-XX:heapDumpPath=heap/heapdump.hprof |                         生成dump文件                         |
|                    -XX:-DoEscapeAnalysis                     |           不使用逃逸分析，jdk6u23之后默认是开启的            |
|                   -XX:+PrintEscapeAnalysis                   |                    查看逃逸分析的筛选结果                    |
|                  -XX:-EliminateAllocations                   |       关闭标量替换，默认开启，需要开启逃逸分析才能关闭       |

通常会将`-Xms`和`-Xmx`两个参数配置相同的值，其目的是为了能够在 java 垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能。

手动生成 dump 文件：

- jmap -dump:format=b,file= <filename.hprof> <pid>
- jmap -dump:live,format=b,file= <filename.hprof> <pid>

### 6. OOM 案例

栈空间溢出 StackOverFlow，一般也不称为 OOM。

- 堆溢出：不断创建对象
- 元空间溢出：不断创建代理类

```java
while (true) {
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(SecurityProperties.User.class);
    enhancer.setUseCache(false);
    enhancer.setCallback(new MethodInterceptor() {
        @Override
        public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
            return proxy.invoke(obj, args);
        }
    });
    Object o = enhancer.create();
}
```

- GC overhead limit exceeded：垃圾回收效率不足，提前报异常
- 线程溢出：创建了大量线程

### 7. JVM 调优

JVM 监控及诊断工具

- 命令行：
  - jps -l

  - jstat -gc <pid> <interval> <print times>

  - jinfo -flag <cared parameter> <pid>

  - jmap

  - jstack <pid> > <log file>

- GUI：Visual VM、eclipse MAT、Arthas

为什么要调优？

- 防止出现 OOM，进行 JVM 规划和预调优
- 解决程序运行中各种 OOM
- 减少 Full GC 出现的频率，解决运行慢、卡顿问题

调优监控的依据：运行日志、异常堆栈、GC 日志、线程快照、堆转储快照

性能优化的步骤：

- 第1步：熟悉业务场景
- 第2步（发现问题）：性能监控：GC 频繁、cpu load 过高、OOM、内存泄漏、死锁、程序响应时间较长
- 第3步（排查问题）：性能分析：
  - 打印GC日志，通过GCviewer或者http://gceasy.io来分析日志信息
  - 灵活运用命令行工具，jstack，jmap，jinfo等
  - dump 出堆文件，使用内存分析工具分析文件
  - 使用阿里 Arthas，或 jconsole，JVisualVM 来实时查看 JVM
  - jstack 查看堆栈信息
- 第4步（解决问题）：性能调优
  - 适当增加内存，根据业务背景选择垃圾回收器
  - 优化代码，控制内存使用
  - 增加机器，分散节点压力
  - 合理设置线程池线程数量
  - 使用中间件提高程序效率，比如缓存，消息队列等
  - 其他......

JIT 在解释运行的时候才会进行优化，所以编译生成的字节码文件不会做**同步消除**（消除无效锁）。

```java
public void function(){
    object o = new Object();
    synchronized(o){...}
}
```

**标量替换：**在 JIT 阶段，如果经过逃逸分析，发现一个对象（那些还可以分解的数据叫做**聚合量** Aggregate）不会被外界访问的话，那么经过 JIT 优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来代替。

**官方推荐设置：**

- Java 整个堆大小设置，Xmx 和 Xms 设置为老年代存活对象的 3-4 倍，即 FullGC 之后的老年代内存占用的 3-4 倍。
- 方法区（永久代 PermSize 和 MaxPermSize 或 元空间 MetaspaceSize 和 MaxMetaspaceSize）设置为老年代存活对象的 1.2-1.5 倍。
- 年轻代Xmn的设置为老年代存活对象的 1-1.5 倍

**CPU 占用很高排查方案：**

- ps aux l grep java 查看到当前 java 进程使用 cpu、内存、磁盘的情况获取使用量异常的进程
- top -Hp <进程pid> 检查当前使用异常线程的 pid
- 把线程 pid 变为 16 进制如 31695 -> 7bcf 然后得到 0x7bcf6，jstack <进程pid> | grep -A20 0x7bcf 得到相关进程的代码

### 8. 垃圾回收

可作为 gc roots 的对象：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象
- 本地方法栈中 JNI（即一般说的 Native 方法）引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象

### 9. HotSpot 垃圾分代回收算法

默认情况下新生代占 1/3，老年代占 2/3

绝大多数对象在新生代中被创建，这里的垃圾回收非常频繁且速度很快

新生代通常采用复制算法，由于存活对象少，复制成本很低

新生代分为 Eden、Surivivor  from、Surivivor to 区，占比 8:1:1

Eden 填满后触发一次新生代的垃圾回收，称为 minor gc，存活对象复制到任一 Surivivor 区，然后将 Eden 区清空即可完成这次 gc，Surivivor from 区的存活对象会复制到另一个 Surivivor to 区，这里需要保证 to 区为空

- 存活超过复制次数阈值（默认 15）会被复制到老年代

- Surivivor 空间不够容纳存活对象时，也会直接进入老年代
- 大数组或者特别大的字符串

老年代通常使用标记整理算法进行回收，将存活对象向一端进行移动，称为 major gc

### 10. G1/CMS 并发标记原理

三色标记：
- 白色：没有被访问过 -> 垃圾对象
- 黑色：包括其引用都被访问过
- 灰色：被访问过，但还存在一些引用没有被访问

**对象消失问题**：扫描过程中插入了一条或多条从黑色对象到白色对象的新引用，并且同时去掉了灰色对象到该白色对象的直接引用或者间接引用。

解决方法，破坏上述两个条件之一即可：

- 增量更新：记录引用关系，并发扫描结束后根据记录重新扫描一次 -> CMS

- 原始快照（SATB）：记录 -> G1

[JVM_垃圾收集之三色标记算法详解](https://blog.csdn.net/chuige2013/article/details/129659171)

CMS 缺点：占用 CPU 资源，不超过 25%；浮动垃圾；内存碎片

### 11. 其他垃圾回收算法

Serial：JDK 1.3 版本之前唯一的串行垃圾回收器，Stop The World

ParNew：多线程垃圾回收，只负责新生代的垃圾回收，可以配合 Serial Old 和 Concurrent Mark Swap 处理老年代

Parallel Scavenge：也是新生代的收集器，可控制吞吐量，gc 自适应，配合 Parallel old 处理老年代

### 12. 类加载机制

类的生命周期：（class 文件 -> Java虚拟机内存 -> 卸载）

- 加载 -> 验证 -> 准备 -> 解析 -> 初始化 -> 使用 -> 卸载

类的加载过程：

- 加载：查找并加载类的二进制数据（Class文件）
  - 方法区：类的类信息
  - 堆：Class 文件对应的类实例
- 验证：确保加载的类信息是正确的
- 准备：为类的静态变量进行初始化，分配空间并赋予初始值。例如：`public static int a = 1;` 在准备阶段对静态变量 a 赋默认值 0
- 解析：是将符号应用转换为直接引用
- 初始化：JVM 对类进行初始化，对静态变量赋予正确值。例如：`public static int a = 1;`这个时候才对静态变量 a 赋初始值 1
  - 静态代码块

### 13. String

String 代表的是 Java 中的字符串 ， String 类比较特殊，它整个类都是被 final 修饰的，也就是说，String 不能被任何类继承，任何修改 String 字符串的⽅法都是创建了⼀个新的字符串（保证了线程安全性）。

不可变对象不是真的不可变，可以通过**反射**来对其内部的属性和值进⾏修改，不过⼀般我们不这样做。  

方法`String.intern()`：在 jdk1.7 及以后调⽤`intern()`⽅法是判断运⾏时常量池中是否有指定的字符串，如果没有的话，就把字符串添加到常量池（jdk1.8 之后，字符串常量池在堆中）中，并返回常量池中的对象。  

```java
String a = new String("ab");
String b = new String("ab");
String c = "ab";
String d = "a";
String e = new String("b");
String f = d + e; // + 号相当于是执行 new StringBuilder.append(), 但每次都会new StringBuilder()，所以多次拼接建议自建 StringBuilder
String g = "a" + "b"; // 编译器会优化，会直接被优化为bbbccc，也就是直接创建了一个 bbbccc 对象

System.out.println(a.intern() == b);          //false
System.out.println(a.intern() == b.intern()); //true
System.out.println(a.intern() == c);          //true
System.out.println(a.intern() == f);	      //false
//equals()方法作对比都是true
```

