### 1. 设计原则

**SOLID 原则：**是面向对象设计和编程中的一组基本原则，由以下五个原则的首字母缩写组成：

- 单一职责原则（Single Responsibility Principle, SRP）：一个类或者模块只应该有一个单一的责任。这个原则告诉我们，一个类应该只负责一项功能，不要试图把太多的职责塞到一个类里面。

- 开闭原则（Open Closed Principle, OCP）：软件应该对扩展开放，对修改关闭。这个原则告诉我们，我们应该尽量通过扩展来实现新的功能，而不是去修改已经存在的代码。

- 里氏替换原则（Liskov Substitution Principle, LSP）：子类可以被看作是父类的一种类型，即父类能出现的地方子类也能够出现。这个原则告诉我们，在使用继承时，子类不能改变父类原有的行为，否则会导致程序出现意想不到的问题。

- 接口隔离原则（Interface Segregation Principle, ISP）：客户端不应该依赖于它不需要的接口。这个原则告诉我们，在设计接口时，应该尽量将接口拆分成更小粒度的接口，避免接口的臃肿和复杂度的增加。

- 依赖倒置原则（Dependency Inversion Principle, DIP）：高层模块不应该依赖低层模块，二者都应该依赖其抽象。这个原则告诉我们，在设计类和模块之间的关系时，应该通过抽象来实现低耦合、高内聚的设计。

### 2. 设计模式的类型

**创建型模式：**这些设计模式提供了一种在创建对象的同时隐藏创建逻辑的方式，而不是使用 new 运算符直接实例化对象。这使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活。包括：工厂模式（Factory Pattern）、抽象工厂模式（Abstract Factory Pattern）、单例模式（Singleton Pattern）、建造者模式（Builder Pattern）、原型模式（Prototype Pattern）

**结构型模式：**这些模式关注对象之间的组合和关系，旨在解决如何构建灵活且可复用的类和对象结构。包括：适配器模式（Adapter Pattern）、桥接模式（Bridge Pattern）、过滤器模式（Filter、Criteria Pattern）、组合模式（Composite Pattern）、装饰器模式（Decorator Pattern）、外观模式（Facade Pattern）、享元模式（Flyweight Pattern）、代理模式（Proxy Pattern）

**行为型模式：** 这些模式关注对象之间的通信和交互，旨在解决对象之间的责任分配和算法的封装。包括：责任链模式（Chain of Responsibility Pattern）、命令模式（Command Pattern）、解释器模式（Interpreter Pattern）、迭代器模式（Iterator Pattern）、中介者模式（Mediator Pattern）、备忘录模式（Memento Pattern）、观察者模式（Observer Pattern）、状态模式（State Pattern）、空对象模式（Null Object Pattern）、策略模式（Strategy Pattern）、模板模式（Template Pattern）、访问者模式（Visitor Pattern）

**J2EE 模式：**这些设计模式特别关注表示层。这些模式是由 Sun Java Center 鉴定的。包括：MVC 模式（MVC Pattern）、业务代表模式（Business Delegate Pattern）、组合实体模式（Composite Entity Pattern）、数据访问对象模式（Data Access Object Pattern）、前端控制器模式（Front Controller Pattern）、拦截过滤器模式（Intercepting Filter Pattern）、服务定位器模式（Service Locator Pattern）、传输对象模式（Transfer Object Pattern）

**并发型模式（Concurrency Patterns）：**这些模式涉及到多线程和并发编程方面的问题，用于解决并发环境下的一些常见问题。常见的并发型模式包括：生产者-消费者模式（Producer-Consumer Pattern）、读者-写者模式（Reader-Writer Pattern）、同步模式（Synchronization Pattern）

### 3. 单例模式

懒汉式单例模式在第一次调用的时候进行实例化。

**适用于单线程环境（不推荐）：**

此方式在单线程的时候工作正常，但在多线程的情况下就有问题了。如果两个线程同时运行到判断instance是否为null的if语句，并且instance的确没有被创建时，那么两个线程都会创建一个实例，此时类型Singleton就不再满足单例模式的要求了。

```java
public class Singleton {
	private static Singleton instance = null;
  
	private Singleton() {}
  
	public static Singleton getInstance() {
		if (null == instance) {
			instance = new Singleton();
		}
		return instance;
	}
}
```

**适用于多线程环境，但效率不高（不推荐）:**

为了保证在多线程环境下我们还是只能得到该类的一个实例，只需要在getInstance()方法加上同步关键字sychronized，就可以了。但每次调用getInstance()方法时都被synchronized关键字锁住了，会引起线程阻塞，影响程序的性能。

```java
public static synchronized Singleton getInstance() {
	if (instance == null) {
		instance = new Singleton();
	}
	return instance;
}
```

**双重检验锁（DCL，即 double-checked locking）：**

为了在多线程环境下，不影响程序的性能，不让线程每次调用`getInstance()`方法时都加锁，而只是在实例未被创建时再加锁，在加锁处理里面还需要判断一次实例是否已存在。

```java
private static volatile Singleton instance = null;

public static Singleton getInstance() {
	// 先判断实例是否存在，若不存在再对类对象进行加锁处理
	if (instance == null) {
		synchronized (Singleton.class) {
			if (instance == null) {
				instance = new Singleton();
			}
		}
	}
	return instance;
}
```

如果 instance 属性不加 volatile，可能会返回未初始化完成的对象。

**静态内部类方式（推荐）：**

加载一个类时，其内部类不会同时被加载。一个类被加载，当且仅当其某个静态成员（静态域、构造器、静态方法等）被调用时发生。 由于在调用`StaticSingleton.getInstance()`的时候，才会对单例进行初始化，而且通过反射，是不能从外部类获取内部类的属性的；由于静态内部类的特性，只有在其被第一次引用的时候才会被加载，所以可以保证其线程安全性。
总结：

- 优势：兼顾了懒汉模式的内存优化（使用时才初始化）以及饿汉模式的安全性（不会被反射入侵）
- 劣势：需要两个类去做到这一点，虽然不会创建静态内部类的对象，但是其 Class 对象还是会被创建，而且是属于永久带的对象

```java
public class StaticSingleton {
	private StaticSingleton() {}
	// 这里可以去掉synchronized，因为JVM会保证静态内部类的加载是线程安全
	public static StaticSingleton getInstance() {
		return StaticSingletonHolder.instance;
	}

	// 一个私有的静态内部类，用于初始化一个静态final实例
	private static class StaticSingletonHolder {
		private static final StaticSingleton instance = new StaticSingleton();
	}
}
```

饿汉式单例类：在类初始化时，已经自行实例化。

**饿汉式：**

```java
public class Singleton {
	private static final Singleton instance = new Singleton();

	private Singleton() {}

	public static Singleton getInstance() {
		return instance;
	}
}
```

**枚举方式（推荐）：**

创建枚举默认就是线程安全的，所以不需要担心 DCL，而且还自动支持序列化机制，防止反序列化重新创建新的对象，绝对防止多次实例化。不能通过 reflection attack 来调用私有构造方法。

```java
public class Singleton {
	enum Single {
		SINGLE;
		private Single() {}

		public void print() {System.out.println("hello world");}
	}
}
```

