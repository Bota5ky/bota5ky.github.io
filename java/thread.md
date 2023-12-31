### 1. 多线程实现方式

**继承 Thread 类，重写 run 方法**

缺点：Java 是单继承的，MyThread 无法再继承其他类

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

**实现 Runnable 接口，重写 run 方法，实现 Runnable 接口的实现类的实例对象作为 Thread 构造函数的 target**

如果要实现多继承就得要用`implements`，Java 提供了接口`java.lang.Runnable`来解决上边的问题。

`Runnable`是可以共享数据的，多个 Thread 可以同时加载一个`Runnable`，当各自 Thread 获得 CPU时间片的时候开始运行`Runnable`，`Runnable`里面的资源是被共享的，所以使用`Runnable`更加的灵活。

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

- 在第二种方法（Runnable）中，ticket 输出的顺序并不是54321，这是因为线程执行的时机难以预测，`ticket--`并不是原子操作。

- 在第一种方法中，我们 new 了3个 Thread 对象，即三个线程分别执行三个对象中的代码，因此便是三个线程去**独立**地完成卖票的任务；而在第二种方法中，我们同样也 new 了3个 Thread 对象，但只有一个`Runnable`对象，3个 Thread 对象共享这个`Runnable`对象中的代码，因此，便会出现3个线程共同完成卖票任务的结果。如果我们 new 出3个`Runnable`对象，作为参数分别传入3个 Thread 对象中，那么3个线程便会**独立**执行各自`Runnable`对象中的代码，即3个线程各自卖5张票。

- 在第二种方法中，由于3个 Thread 对象共同执行一个`Runnable`对象中的代码，因此可能会造成线程的不安全，比如可能 ticket 会输出-1（如果我们`System.out....`语句前加上线程休眠操作，该情况将很有可能出现），这种情况的出现是由于，一个线程在判断 ticket 为1>0后，还没有来得及减1，另一个线程已经将 ticket 减1，变为了0，那么接下来之前的线程再将 ticket 减1，便得到了-1。这就需要加入同步操作（即互斥锁），确保同一时刻只有一个线程在执行每次for循环中的操作。而在第一种方法中，并不需要加入同步操作，因为每个线程执行自己 Thread 对象中的代码，不存在多个线程共同执行同一个方法的情况。

**通过 Callable 和 FutureTask 创建线程**

`Runnable`是执行工作的独立任务，但是它不返回任何值。如果你希望任务在完成的能返回一个值，那么可以实现`Callable`接口而不是`Runnable`接口。在 Java SE5 中引入的`Callable`是一种具有类型参数的泛型，它的参数类型表示的是从方法`call()`(不是`run()`)中返回的值。

`FutureTask`具有仅执行1次`run()`方法的特性(**即使有多次调用也只执行1次**)，避免了重复查询的可能。

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

**通过线程池创建线程**

```java
ExecutorService executorService = Executors.newFixedThreadPool(5);
MyRunnable myRunable = new MyRunnable();
executorService.execute(myRunable);
executorService.shutdown();
```

- `ExecutorService`、`Callable`、`Future`三个接口实际上都是属于`Executor`框架。返回结果的线程是在 JDK 1.5 中引入的新特征，有了这种特征就不需要再为了得到返回值而大费周折了

- 有返回值的任务必须实现`Callable`接口。类似的，无返回值的任务必须实现`Runnable`接口

- 执行`Callable`任务后，可以获取一个`Future`的对象，在该对象上调用`get`就可以获取到`Callable`任务返回的`Object`了

- 注意：`get`方法是阻塞的，即：线程无返回结果，`get`方法会一直等待

- 再结合线程池接口`ExecutorService`就可以实现传说中有返回结果的多线程了

- 下面提供了一个完整的有返回结果的多线程测试例子，在 JDK 1.5 下验证过没问题可以直接使用


### 2. 实现 Runnable 接口相比继承 Thread 类的优势

- 可以避免由于 Java 的单继承特性而带来的局限

- 增强程序的健壮性，代码能够被多个线程共享，代码与数据是独立的

- 适合多个相同程序代码的线程区处理同一资源的情况


### 3. 实现 Runnable 接口和实现 Callable 接口的区别

- `Runnable`是自从 Java 1.1 就有了，而`Callable`是 1.5 之后才加上去的

- `Callable`规定的方法是`call()`，`Runnable`规定的方法是`run()`

- `Callable`的任务执行后可返回值，而`Runnable`的任务是不能返回值（是 void）

- `call`方法可以抛出异常，`run`方法不可以
- 运行`Callable`任务可以拿到一个`Future`对象，表示异步计算的结果。它提供了检查计算是否完成的方法，以等待计算的完成，并检索计算的结果。通过`Future`对象可以了解任务执行情况，可取消任务的执行，还可获取执行结果
- 加入线程池运行，`Runnable`使用`ExecutorService`的`execute`方法，`Callable`使用`submit`方法

### 4. 线程的六种状态

- NEW：初始化状态，未调用`start`方法
- RUNNABLE：运行态
  - Ready：等待CPU发时间片
  - Running：执行态
- BLOCKED：阻塞态，锁
- WAITING：等待态，拿到锁之后，发现当前执行条件不太满足，需要暂时停止执行，以便让出CPU资源供其他线程执行，`wait()`，继续执行需要`notify()`
- TIMED_WAITING：超时等待
- TERMINATED：结束态

### 5. sleep 和 wait 的区别

sleep
- 暂停当前线程执行，但不释放锁
- `Thread.sleep()`
- 可以在任何场景使用
- 只能等待 sleep 结束

wait
- 暂停当前线程执行，并释放锁
- 属于对象
- 不能写在`synchronized`同步块之外
- `notify()`唤醒

### 6. 停止线程

但是`stop()`方法是不建议使用，并且是有可能在未来版本中删除掉的：

```java
@Deprecated(since="1.2", forRemoval=true)
public final void stop() {}
```

因为`stop()`方法太粗暴了，一旦调用了`stop()`，就会**直接停掉线程**，这样就可能造成严重的问题，比如任务执行到哪一步了？该释放的锁释放了没有？都存在疑问。

> 这里强调一点，`stop()`会释放线程占用的`synchronized`锁，而不会自动释放`ReentrantLock`锁

建议通过中断来停止线程`t1.interrupt()`，执行结果根据线程内部的逻辑决定

```java
if (Thread.currentThread().isInterrupted() && otherOptions) {
    //中断执行的操作break
}

try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    //如果没有对中断做处理，中断信号会被重置为false，在循环内判断isInterrupted就会失效
}
```

线程池也是通过 interrupt 方法来停止线程的，比如 shutdownNow() 方法中会调用：

```java
void interruptIfStarted() {
	Thread t;
	if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
		try {
			t.interrupt();
		} catch (SecurityException ignore) {
		}
	}
}
```

### 7. 有序性

Java JIT（Just In Time）和 CPU 都有可能对指令做重排序。

对于懒汉式的单例模式，在多线程的情况下可能会存在问题，因为 B 线程初始化单例实际分为 3 个指令：申请空间、初始化、赋值，后面 2 个指令有可能会被调换顺序。当 B 线程先执行了赋值，还未进行初始化，此时暂停 B 线程转而继续执行 A 线程，就会导致 A 线程直接返回未初始化完成的 instance。解决方法就是给 instance 变量加上关键字 volatile。

```java
public static Singleton getInstance() { //A
	// 先判断实例是否存在，若不存在再对类对象进行加锁处理
	if (instance == null) {
		synchronized (Singleton.class) {
			if (instance == null) {
				instance = new Singleton(); //B
			}
		}
	}
	return instance;
}
```

### 8. ThreadLocal

- `ThreadLocal`是 Java 中所提供的线程本地存储机制，可以利用该机制将数据**缓存在某个线程内部**，该线程可以在任意时刻、任意方法中获取缓存的数据。

- `ThreadLocal`底层是通过`ThreadLocalMap`来实现的，每个`Thread`对象（注意不是`ThreadLocal`对象）中都存在一个`ThreadLocalMap`，Map 的 key 为`ThreadLocal`对象，Map 的 value 为需要缓存的值。

```java
// Thread.java
ThreadLocal.ThreadLocalMap threadLocals = null;
```

- 如果在线程池中使用`ThreadLocal`会造成**内存泄漏**，因为当`ThreadLocal`对象使用完之后，应该要把设置的 key、value，也就是`Entry`对象进行回收，但线程池中的线程不会回收，而线程对象是通过强引用指向`ThreadLocalMap`，`ThreadLocalMap`也是通过强引用指向`Entry`对象线程不被回收，`Entry`对象也就不会被回收，从而出现内存泄漏，解决办法是，在使用了`ThreadLocal`对象之后，手动调用`ThreadLocal`的`remove`方法，手动清除`Entry`对象。

```java
threadLocal.remove();
```

`ThreadLocal`经典的应用场景就是连接管理（一个线程持有一个连接，该连接对象可以在不同的方法之间进行传递，线程之间不共享同一个连接）。

### 9. fail-fast 和 fail-safe   

- fail-fast是 Java 中的⼀种**快速失败**机制， java.util 包下所有的集合都是快速失败的，快速失败会抛出 `ConcurrentModificationException`异常， fail-fast 你可以把它理解为⼀种快速检测机制，它只能⽤来检测错误，不会对错误进⾏恢复， fail-fast 不⼀定只在**多线程**环境下存在， ArrayList 也会抛出这个异常，主要原因是由于 modCount 不等于 expectedModCount。
- fail-safe是 Java 中的⼀种**安全失败**机制，它表示的是在遍历时不是直接在原集合上进⾏访问，而是先复制原有集合内容，在拷⻉的集合上进⾏遍历。 由于迭代时是对原集合的拷⻉进⾏遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发`ConcurrentModificationException`。`java.util.concurrent`包下的容器都是安全失败的，可以在多线程条件下使⽤，并发修改。  

### 10. CTL (control)

在 Java 中，线程池（Thread Pool）是一种用于管理和复用线程的机制。在 Java 的线程池实现中，`ctl`是一个表示线程池状态和线程数量的变量。

具体来说，`ctl`是一个 32 位的整数，其中高 3 位表示线程池的状态，低 29 位表示线程池中的线程数量。这样的设计可以同时表示线程池的状态和线程数量，提供了一种紧凑的表示方式。

通过对`ctl`的操作，可以实现对线程池状态和线程数量的管理，包括增加或减少线程数量、判断线程池是否在运行等功能。这样的设计可以更好地控制线程池的行为和资源利用，提高多线程编程的效率和可靠性。

### 11. 线程池的创建

```java
//三大方法
Executors.newSingleThreadExecutor(); // 单个线程
Executors.newFixedThreadPool(5); // 固定的线程池大小
Executors.newCachedThreadPool(); // 可伸缩的
//七大参数
public ThreadPoolExecutor(
	int corePoolSize,
	int maximumPoolSize,
	long keepAliveTime,
	TimeUnit unit,	// TimeUnit.SECONDS
	BlockingQueue<Runnable> workQueue,	// new LinkedBlockingDeque<>(3)
	ThreadFactory threadFactory,	// Executors.defaultThreadFactory()
	RejectedExecutionHandler handler // new ThreadPoolExecutor.AbortPolicy()
)
```

### 12. 线程池的状态

| 状态       | 行为                                                         |
| :--------- | :----------------------------------------------------------- |
| RUNNING    | **会**接收新任务并且**会**处理队列中的任务                   |
| SHUTDOWN   | **不会**接收新任务并且**会**处理队列中的任务，任务处理完后会中断所有线程 |
| STOP       | **不会**接收新任务并且**不会**处理队列中的任务，并且会直接中断 |
| TIDYING    | 所有线程所有线程都停止了之后，线程池的状态就会转为TIDYING，一旦达到此状态，就会调用线程池的`terminated()` |
| TERMINATED | `terminated()`执行完之后就会转变为TERMINATED                 |

`SHUTDOWN`对应 executor.shutdown() 方法，`STOP`对应 executor.shutdownNow() 方法

线程池状态的转换情况

|  转变前  |   转变后   |                           转变条件                           |
| :------: | :--------: | :----------------------------------------------------------: |
| RUNNING  |  SHUTDOWN  | 手动调用`shutdown()`触发，或者线程池对象GC时会调用`finalize()`从而调用`shutdown()` |
| RUNNING  |    STOP    |                 手动调用`shutdownNow()`触发                  |
| SHUTDOWN |    STOP    |      手动先调用`shutdown()`紧着调用`shutdownNow()`触发       |
| SHUTDOWN |  TIDYING   |                线程池所有线程都停止后自动触发                |
|   STOP   |  TIDYING   |                线程池所有线程都停止后自动触发                |
| TIDYING  | TERMINATED |              线程池自动调用`terminated()`后触发              |

### 13. 线程池中提交一个任务的流程

<center><img src="threadpool.svg" style="zoom:120%"></center>

### 14. 四种拒绝策略

- AbortPolicy 如果线程池拒绝了任务，直接报错
- CallerRunsPolicy 线程池让调用者去执行

```java
public static class CallerRunsPolicy implements RejectedExecutionHandler {
	public CallerRunsPolicy() { }
    public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
		if (!e.isShutdown()) {
			r.run(); // 可以看见源码逻辑为，先判断线程池还未关闭，然后直接r.run运行了任务。
		}
	}
}
```

- DiscardPolicy 如果线程池拒绝了任务，直接丢弃
- DiscardOldestPolicy 如果线程池拒绝了任务，直接将线程池中最旧的，未运行的任务丢弃，将新任务入队

### 15. 为什么不建议使用 Executors 来创建线程池？

当我们使用 Executors 创建 FixedThreadPool 时，对应的构造方法为：

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,
        new LinkedBlockingQueue<Runnable>());
}
```

发现创建的队列为 LinkedBlockingQueue，是一个无界阻塞队列，如果使用该线程池执行任务如果任务过多就会不断的添加到队列中，任务越多占用的内存就越多，最终可能耗尽内存，导致OOM。

> 阿里开发手册：线程池不允许使用 Executors 去创建， 而是通过 ThreadPoolExecutor 的方式， 这样的处理方式让写的同学更加明确线程池的运行规则， 规避资源耗尽的风险。
> Executors 返回的线程池对象的弊端如下：
>
> - FixedThreadPool 和 SingleThreadPool：允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。
> - CachedThreadPool：允许的创建线程数量为 Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。
> - ScheduledThreadPool：允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM。 

除开有可能造成 OOM 之外，我们使用 Executors 来创建线程池也不能自定义线程的名字，不利于排查问题，所以建议直接使用 ThreadPoolExecutor来定义线程池，这样可以灵活控制。

