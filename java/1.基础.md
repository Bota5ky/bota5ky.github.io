### JDK, JRE, JVM

JDK(Java Development Kit)

- 开发工具
  - 基本类库
  - javac 编译
  - javap 反编译
  - javadoc
- 运行环境 JRE(Java Runtime Environment)
  - JVM(Java Virtual Mechinal) 不同操作系统的机器指令是有可能不一样的，所以就导致不同操作系统上的JVM是不一样的。凡是编译后是Java字节码的都可以在JVM上运行，如Apache Groovy，Scala and Kotlin等等。
  - JVM工作所需要的类库

### Math.Round()

向右取整

### Byte范围

Java 中用`补码`来表示⼆进制数，补码的最高位是符号位，最高位用`0`表示正数，最高位`1`表示负数，正数的补码就是其`本身`，由于最高位是符号位，所以正数表示的就是`0111 1111`，也就是`127`。最⼤负数就是`1111 1111`，这其中会涉及到两个`0`，⼀个`+0`，⼀个`-0`，`+0`归为正数，也就是`0`，`-0`归为负数，也就是`-128`，所以 byte 的范围就是 -128 - 127。

### Integer 缓存池

它的默认值⽤于缓存 -128 - 127 之间的数字，如果有 -128 - 127 之间的数字的话，使⽤ new Integer
不⽤创建对象，会直接从缓存池中取，此操作会减少堆中对象的分配，有利于提⾼程序的运⾏效率。

例如创建⼀个 Integer a = 24，其实是调⽤ Integer 的 valueOf

### UTF-8 和 Unicode 的关系  

由于每个国家都有⾃⼰独有的字符编码，所以Unicode 的发展旨在创建⼀个新的标准，⽤来映射当今使用的大多数语⾔中的字符，这些字符有⼀些不是必要的，但是对于创建⽂本来说却是不可或缺的。
Unicode 统⼀了所有字符的编码，是⼀个 Character Set，也就是字符集，字符集只是给所有的字符⼀个唯⼀编号，但是却没有规定如何存储，不同的字符其存储空间不⼀样，有的需要⼀个字节就能存储，有的则需要2、 3、 4个字节。
UTF-8 只是众多能够对⽂本字符进行解码的⼀种⽅式，它是⼀种变长的方式。 UTF-8 代表 8 位⼀组表示 Unicode 字符的格式，使⽤ 1 - 4 个字节来表示字符。

```markdown
U+ 0000 ~ U+ 007F: 0XXXXXXX
U+ 0080 ~ U+ 07FF: 110XXXXX 10XXXXXX
U+ 0800 ~ U+ FFFF: 1110XXXX 10XXXXXX 10XXXXXX
U+10000 ~ U+1FFFF: 11110XXX 10XXXXXX 10XXXXXX 10XXXXXX
```

可以看到， UTF-8 通过开头的标志位位数实现了变⻓。对于单字节字符，只占⽤⼀个字节，实现了向下兼容 ASCII，并且能和 UTF-32 ⼀样，包含 Unicode 中的所有字符，⼜能有效减少存储传输过程中占⽤的空间。  

*char* 在 java 中是2个字节。 java 采用 unicode，2个字节（16位）来表示一个字符。所以 char = '中' 合法。

### fail-fast 和 fail-safe   

- `fail-fast`是 Java 中的⼀种`快速失败`机制， java.util 包下所有的集合都是快速失败的，快速失败会抛出 `ConcurrentModificationException`异常， fail-fast 你可以把它理解为⼀种快速检测机制，它只能⽤来检测错误，不会对错误进⾏恢复， fail-fast 不⼀定只在`多线程`环境下存在， ArrayList 也会抛出这个异常，主要原因是由于 modCount 不等于 expectedModCount。
- `fail-safe`是 Java 中的⼀种`安全失败`机制，它表示的是在遍历时不是直接在原集合上进⾏访问，而是先复制原有集合内容，在拷⻉的集合上进⾏遍历。 由于迭代时是对原集合的拷⻉进⾏遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发ConcurrentModificationException。`java.util.concurrent`包下的容器都是安全失败的，可以在多线程条件下使⽤，并发修改。  

### 反射的基本原理，反射创建类实例的三种方式是什么

反射机制就是使 Java 程序在运⾏时具有 自省(introspect) 的能力，通过反射我们可以直接操作类和对象，比如获取某个类的定义，获取类的属性和方法，构造方法等。
创建类实例的三种⽅式是：

- 对象实例.getClass()
- 通过 Class.forName() 创建
- 对象实例.newInstance() ⽅法创建  

### 类加载机制

类的生命周期：（class 文件 -> Java虚拟机内存 -> 卸载）

- 加载
- 验证
- 准备
- 解析
- 初始化
- 使用
- 卸载

类的加载过程：

- 加载：查找并加载类的二进制数据（Class文件）
  - 方法区：类的类信息
  - 堆：Class 文件对应的类实例
- 验证：确保加载的类信息是正确的
- 准备：为类的静态变量进行初始化，分配空间并赋予初始值
- 解析：是将符号应用转换为直接引用
- 初始化：JVM对类进行初始化，对静态变量赋予正确值
  - 静态代码块

类加载器：

- BootStrapClasserLoader C语言实现，加载 JDK/JRE/lib 下类路径 java.* 开头的类
- ExtClassLoader 扩展类加载器 加载 JDK/JRE/lib/ext 下类路径 javax.* 开头的类
- AppClassLoader 加载自己定义的类，类路径下面。用户自定义类加载器 可以用流、文件、数据库、网络形式加载

### 双亲委派模型

当一个类加载器收到了类加载的请求的时候，他不会直接去加载指定的类，而是把这个请求委托给自己的父加载器去加载。只有父加载器无法加载这个类的时候，才会由当前这个加载器来负责类的加载。（这样就保证了我们不能篡改 java.己有的类）

### 异常

**参考文献：[看完这篇 Exception 和 Error ，和面试官扯皮就没问题了](https://mp.weixin.qq.com/s?__biz=MzkwMDE1MzkwNQ==&mid=2247495984&idx=1&sn=97c90237ae80d1a244bd34edf2628986&source=41#wechat_redirect)**

Java中的所有异常都来自顶级父类 Throwable（实现 Serializable 接口）。

Throwable下有两个子类Exception和Error。

- Eror表示非常严重的措误，比 StakOverFlowError 和 0ut0fMemoryError，通常这些误出现时，仅仅想靠程序自己是解决不了的，可能是虚拟机、磁盘、操作系统层面出现的问题了，所以通常也不建议在代码中去捕获这些Eror，因为捕获的意义不大，因为程序可能已经根本运行不了了。
- Exception表示异常，表示程序出现 Exception时，是可以靠程序自己来解决的，比 NullPointerException、 IllealAcessException等，我们可以捕获这些异常来做特殊处理。Exception的子类通常又可以分为RuntimeException和非RuntimeException两类。
  - RuntimeException 表示运行期异常（又称 uncheckedException），表示这个异常是在代码运行过程中抛出的，这些异常是非检查异常，程序中可以选泽浦获处理，也可以不处理，这些导一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生，比如 NullPointerException、IndexOutOfBoundsException 等
  - 非RuntimeException表示非运行期异常，也就是我们常说检查异常，是必须进行处的异常，如果不处理，程序就不能检查异常通过，如 IOExeption、SQLException 等以及用户自定义的 Exception 异常

### String

String 代表的是 Java 中的字符串 ， String 类比较特殊，它整个类都是被 final 修饰的，也就是说，String 不能被任何类继承，任何 修改 String 字符串的⽅法都是创建了⼀个新的字符串（保证了线程安全性）。

不可变对象不是真的不可变，可以通过 反射 来对其内部的属性和值进⾏修改，不过⼀般我们不这样做。  

方法 `String.intern()  `：在 JDK1.7 及以后调⽤ intern ⽅法是判断运⾏时常量池中是否有指定的字符串，如果没有的话，就把字符串添加到常量池（JDK8之后，字符串常量池在堆中）中，并返回常量池中的对象。  

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

**具体分析：[一篇与众不同的 String、StringBuilder 和 StringBuffer 详解](https://mp.weixin.qq.com/s?__biz=MzI0ODk2NDIyMQ==&mid=2247484794&idx=1&sn=22efd808fa5a9e68cacabd4b6e08fdc3&chksm=e999f068deee797eef9b46b160c06afa4d50e03b3626d1ae1aad05ddc37ec9001c4514264e0f&token=1065926980&lang=zh_CN%23rd)**

- StringBuilder 不加锁
- StringBuffer 线程安全

### 为什么重写 equals 方法必须重写 hashcode 方法  

- 如果在 Java 运⾏期间对同⼀个对象调用 hashCode 方法后，无论调用多少次，都应该返回相同的 hashCode，但是在不同的 Java 程序中，执行 hashCode 方法返回的值可能不⼀致；
- 如果两个对象的 equals 相等，那么 hashCode 必须相同；
- 如果两个对象 equals 不相等，那么 hashCode 也有可能相同，所以需要重写 hashCode 方法，这样也能够提高不同对象的访问速度；
- hashCode 通常是将地址转换为整数来实现的。  

### String s1 = new String("abc") 在内存中创建了几个对象  

⼀个或者两个，String s1 是声明了⼀个 String 类型的 s1 变量，它不是对象。使⽤ new 关键字会在堆中创建⼀个对象，另外⼀个对象是 abc ，它会在常量池中创建，所以⼀共创建了两个对象；如果 abc 在常量池中已经存在的话，那么就会创建⼀个对象。  

### 设计原则

#### SOLID原则是面向对象设计和编程中的一组基本原则，其中SOLID分别是以下五个原则的首字母缩写：

- 单一职责原则(Single Responsibility Principle，SRP)。一个类或者模块只应该有一个单一的责任。这个原则告诉我们，一个类应该只负责一项功能，不要试图把太多的职责塞到一个类里面。

- 开闭原则(Open Closed Principle，OCP)。软件应该对扩展开放，对修改关闭。这个原则告诉我们，我们应该尽量通过扩展来实现新的功能，而不是去修改已经存在的代码。

- 里氏替换原则(Liskov Substitution Principle，LSP)。子类可以被看作是父类的一种类型，即父类能出现的地方子类也能够出现。这个原则告诉我们，在使用继承时，子类不能改变父类原有的行为，否则会导致程序出现意想不到的问题。

- 接口隔离原则(Interface Segregation Principle，ISP)。客户端不应该依赖于它不需要的接口。这个原则告诉我们，在设计接口时，应该尽量将接口拆分成更小粒度的接口，避免接口的臃肿和复杂度的增加。

- 依赖倒置原则(Dependency Inversion Principle，DIP)。高层模块不应该依赖低层模块，二者都应该依赖其抽象。这个原则告诉我们，在设计类和模块之间的关系时，应该通过抽象来实现低耦合、高内聚的设计。

#### 开闭原则

开闭原则是指软件设计中的一个基本原则，它强调"软件实体（类、模块、函数等）应该对扩展开放，对修改关闭"。换言之，开闭原则要求我们在设计软件时，应该尽量避免直接修改已有的代码，而是通过添加新的代码来扩展功能，从而使系统更加稳定和灵活。

这个原则的核心思想就是面向对象设计的继承和多态特性，即通过继承来扩展原有的功能，而不是直接修改原有的代码。同时，通过多态可以将具体的实现与抽象的接口分离开来，从而降低了代码的耦合度，提高了代码的可维护性和可扩展性。

总之，遵循开闭原则可以使软件系统具有更好的可维护性、可扩展性和可复用性，从而降低软件开发的成本和风险。

#### 里氏替换原则

里氏替换原则是面向对象设计中的重要原则之一，它指出：任何一个基类可以出现的地方，子类一定可以出现。也就是说，子类可以完全替代父类并且不会影响程序的正确性。

这个原则的意义在于保证代码的可维护性、可扩展性和可复用性。如果不遵循里氏替换原则，可能会导致程序的耦合度过高，增加了后期维护的难度，并且给系统带来了潜在风险。

具体而言，里氏替换原则需要满足以下条件：

1. 子类必须完全实现父类的抽象方法。

2. 子类可以有自己的方法，但不能覆盖父类的非抽象方法。

3. 子类的前置条件必须弱于父类；子类的后置条件必须强于父类。

4. 子类不能抛出比父类更多或更宽泛的异常。

总之，里氏替换原则是一种优秀的编程习惯，它可以帮助我们编写出高质量、可维护的代码，提高程序的灵活性和可复用性。

#### 接口隔离原则

接口隔离原则（Interface Segregation Principle，ISP）是面向对象设计中的一项原则，指的是客户端不应该依赖于它不需要的接口。简而言之，一个接口应该只包含客户端需要的方法，而不应该强迫客户端实现它们不需要的方法。

这个原则的目标是减少系统的耦合性，提高系统的可维护性、可扩展性和可重用性。如果一个接口包含了过多的方法，那么实现这个接口的类就会出现“胖接口”的问题，这样会导致代码的臃肿和复杂度的增加，影响程序的可读性、可维护性和可测试性。

因此，按照接口隔离原则，我们应该将一个大接口拆分成多个小接口，每个小接口提供一组相关的方法，客户端只需要实现自己需要的接口即可。这样可以降低实现的难度，减少出错的可能性，同时也方便后期对系统的维护和修改。

#### 依赖倒置原则

依赖倒置原则（Dependence Inversion Principle，DIP）是指设计代码结构时，高层模块不应该依赖低层模块，二者都应该依赖其抽象。抽象不应该依赖细节，细节应该依赖抽象。这个原则是面向对象设计中很重要的一条原则之一，它有助于降低代码的耦合性和提高代码的灵活性、可读性和可扩展性。