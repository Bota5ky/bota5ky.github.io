Java默认有2个线程：`main` + `GC`

并发：CPU单核，交替执行

并行：CPU多核，多个线程可以同时执行（提高使用效率：线程池）

```java
Runtime.getRuntime().availableProcessors() //当前CPU可用核数
```

### 多线程实现方式

#### 继承 Thread 类，重写 run 方法

这样代码的写法简单，符合大家的习惯，但是直接继承Thread类有一个很大的缺点，因为“java类的继承是单一的，extends后面只能指定一个父类”，所有如果当前类继承Thread类之后就不可以继承其他类。如果我们的类已经从一个类继承（如Swing继承自 Panle 类、JFram类等），则无法再继承 Thread 类，这时如果我们又不想建立一个新的类，应该怎么办呢？

```java
class MyThread extends Thread {
    private int ticket = 5;
    
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			if (ticket > 0) {
				System.out.println("车票第" + ticket-- + "张");
			}
		}
	}
}

public void Test() {
	new MyThread().start();
    new MyThread().start();
    new MyThread().start();
}
```

#### 实现 Runnable 接口，重写 run 方法，实现 Runnable 接口的实现类的实例对象作为 Thread 构造函数的 target

如果要实现多继承就得要用implements，Java 提供了接口 java.lang.Runnable来解决上边的问题。

Runnable是可以共享数据的，多个Thread可以同时加载一个Runnable，当各自Thread获得CPU时间片的时候开始运行Runnable，Runnable里面的资源是被共享的，所以使用Runnable更加的灵活。

```java
class MyRunnable implements Runnable {
    private int ticket = 5;
    
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			if (ticket > 0) {
				System.out.println("车票第" + ticket-- + "张");
			}
		}
	}
}

public void Test() {
	Runnable myRunnable = new MyRunnable();
	// 将myRunnable作为Thread target创建新的线程
	new Thread(myRunnable).start();
	new Thread(myRunnable).start();
}
```

1. 在第二种方法（Runnable）中，ticket输出的顺序并不是54321，这是因为线程执行的时机难以预测，`ticket--`并不是原子操作。
2. 在第一种方法中，我们new了3个Thread对象，即三个线程分别执行三个对象中的代码，因此便是三个线程去**独立**地完成卖票的任务；而在第二种方法中，我们同样也new了3个Thread对象，但只有一个Runnable对象，3个Thread对象共享这个Runnable对象中的代码，因此，便会出现3个线程共同完成卖票任务的结果。如果我们new出3个Runnable对象，作为参数分别传入3个Thread对象中，那么3个线程便会**独立**执行各自Runnable对象中的代码，即3个线程各自卖5张票。
3. 在第二种方法中，由于3个Thread对象共同执行一个Runnable对象中的代码，因此可能会造成线程的不安全，比如可能ticket会输出-1（如果我们System.out....语句前加上线程休眠操作，该情况将很有可能出现），这种情况的出现是由于，一个线程在判断ticket为1>0后，还没有来得及减1，另一个线程已经将ticket减1，变为了0，那么接下来之前的线程再将ticket减1，便得到了-1。这就需要加入同步操作（即互斥锁），确保同一时刻只有一个线程在执行每次for循环中的操作。而在第一种方法中，并不需要加入同步操作，因为每个线程执行自己Thread对象中的代码，不存在多个线程共同执行同一个方法的情况。

#### 通过 Callable 和 FutureTask 创建线程

Runnable是执行工作的独立任务，但是它不返回任何值。如果你希望任务在完成的能返回一个值，那么可以实现Callable接口而不是Runnable接口。在Java SE5中引入的Callable是一种具有类型参数的泛型，它的参数类型表示的是从方法`call()`(不是`run()`)中返回的值。

FutureTask具有仅执行1次run()方法的特性(**即使有多次调用也只执行1次**)，避免了重复查询的可能。

```java
class MyThread implements Callable<Integer> {
	public Integer call() throws Exception {
        System.out.println("当前线程名——" + Thread.currentThread().getName());
        int i = 0;
        for (; i < 5; i++) {
            System.out.println("循环变量i的值：" + i);
        }

        return i;
    }
}

public void Test() {
    MyThread myThread = new MyThread();
	FutureTask<Integer> futureTask = new FutureTask<>(myThread);
	new Thread(futureTask, "线程名：有返回值的线程2").start();
	try {
		System.out.println("子线程的返回值：" + futureTask.get());
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```

#### 通过线程池创建线程

```java
ExecutorService executorService = Executors.newFixedThreadPool(5);
MyRunnable myRunable = new MyRunnable();
executorService.execute(myRunable);
executorService.shutdown();
```

1. ExecutorService、Callable、Future三个接口实际上都是属于Executor框架。返回结果的线程是在JDK1.5中引入的新特征，有了这种特征就不需要再为了得到返回值而大费周折了。

2. 有返回值的任务必须实现Callable接口。类似的，无返回值的任务必须实现Runnable接口。

3. 执行Callable任务后，可以获取一个Future的对象，在该对象上调用get就可以获取到Callable任务返回的Object了。

4. 注意：get方法是阻塞的，即：线程无返回结果，get方法会一直等待。

5. 再结合线程池接口ExecutorService就可以实现传说中有返回结果的多线程了。

6. 下面提供了一个完整的有返回结果的多线程测试例子，在JDK1.5下验证过没问题可以直接使用。


### 实现Runnable接口相比继承Thread类有如下优势：

1. 可以避免由于Java的单继承特性而带来的局限；
2. 增强程序的健壮性，代码能够被多个线程共享，代码与数据是独立的；
3. 适合多个相同程序代码的线程区处理同一资源的情况。


### 实现Runnable接口和实现Callable接口的区别:

1. Runnable是自从java1.1就有了，而Callable是1.5之后才加上去的
2. Callable规定的方法是call(),Runnable规定的方法是run()
3. Callable的任务执行后可返回值，而Runnable的任务是不能返回值(是void)
4. call方法可以抛出异常，run方法不可以
5. 运行Callable任务可以拿到一个Future对象，表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。通过Future对象可以了解任务执行情况，可取消任务的执行，还可获取执行结果。
6. 加入线程池运行，Runnable使用ExecutorService的execute方法，Callable使用submit方法。

### 线程的六种状态

- NEW：初始化状态，未调用start方法
- RUNNABLE：运行态
  - Ready：等待CPU发时间片
  - Running：执行态
- BLOCKED：阻塞态，锁
- WAITING：等待态，拿到锁之后，发现当前执行条件不太满足，需要暂时停止执行，以便让出CPU资源供其他线程执行，wait()，继续执行需要notify()
- TIMED_WAITING：超时等待
- TERMINATED：结束态

### Sleep 和 wait 的区别

- Sleep
  - 暂停当前线程执行，但不释放锁
  - Thread.Sleep()
  - 可以在任何场景使用
  - 只能等待sleep结束
- wait
  - 暂停当前线程执行，并释放锁
  - 属于对象
  - 不能写在 synchronized 同步块之外
  - notify() 唤醒

### 停止线程

但是stop()方法是不建议使用，并且是有可能在未来版本中删除掉的：

```java
@Deprecated(since="1.2", forRemoval=true)
public final void stop() {
```

因为stop()方法太粗暴了，一旦调用了stop()，就会**直接停掉线程**，这样就可能造成严重的问题，比如任务执行到哪一步了？该释放的锁释放了没有？都存在疑问。

> 这里强调一点，stop()会释放线程占用的synchronized锁，而不会自动释放ReentrantLock锁

建议通过中断来停止线程

```java
if (Thread.currentThread().isInterrupted()) { //... }
    
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    //如果没有对中断做处理，中断信号会被重置为false，在循环内判断isInterrupted就会失效
}
```



### 线程的优先级越高越先执行吗？

Java线程

- 轻量级进程 -> 系统内核线程，即操作系统原生的线程
- 优先级：1-10，win系统只有7级
- 系统会改变优先级，win存在优先级推进器的功能

### CAS(Compare And Swap)

- 内存位置，预期值，新值
- 原子操作，CPU硬件指令集提供，硬件保证一个语义看起来需要多次操作才能完成的行为，通过一条处理器指令CAS就能完成
- JDK1.5之后提供了CAS操作 Unsafe类

### volatile

- volatile关键字主要是使变量在多个线程间可见

- 线程的私有堆栈：Java内存模型告诉我们，各个线程会将共享变量从主内存中拷贝到工作内容，然后执行引擎会基于工作内存中的数据进行操作处理。这个时机对普通变量是没有规定的，而针对volatile修饰的变量给java虚拟机特殊的约定，线程对volatile变量的修改会立刻被其他线程所感知，即不会出现数据脏读的现象，从而保证数据的”可见性“。

- 只保证可见性，并不保证原子性

### CountDownLatch 减计数器

https://www.cnblogs.com/cxuanBlog/p/14166322.html

### CyclicBamer 加计数器

https://www.jianshu.com/p/4ef4bbf01811

### Semaphore 信号量

https://www.jianshu.com/p/38630b7dbe73

### BlockingQueue 四组API

|    方式    | 抛出异常 | 有返回值，不抛出异常 | 阻塞等待 | 超时等待 |
| :--------: | :------: | :------------------: | :------: | :------: |
|    添加    |   add    |        offer         |   put    |  offer   |
|    移除    |  remove  |         poll         |   take   |   poll   |
| 判断队列首 | element  |         peek         |    /     |    /     |

### ThreadLocal

ThreadLocal 是 Java 中所提供的线程本地存储机制，可以利用该机制将数据缓存在某个线程内部，该线程可以在任意时刻、任意方法中获取缓存的数据。

ThreadLocal 底层是通过 ThreadLocalMap 来实现的，每个 Thread 对象（注意不是 ThreadLocal 对象）中都存在一个 ThreadLocalMap，Map 的 key 为 ThreadLocal 对象，Map 的 value 为需要缓存的值。

如果在线程池中使用 ThreadLocal 会造成**内存泄漏**，因为当 ThreadLocal 对象使用完之后，应该要把设置的 key、value，也就是Entry对象进行回收，但线程池中的线程不会回收，而线程对象是通过强引用指向ThreadLocalMap，ThreadLocalMap 也是通过强引用指向 Entry 对象线程不被回收，Entry 对象也就不会被回收，从而出现内存泄漏，解决办法是，在使用了 ThreadLocal 对象之后，手动调用 ThreadLocal 的 remove 方法，手动清除Entry对象。

```java
threadLocal.remove();
```

ThreadLocal 经典的应用场景就是连接管理（一个线程持有一个连接，该连接对象可以在不同的方法之间进行传递，线程之间不共享同一个连接）。