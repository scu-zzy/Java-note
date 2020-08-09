# 1.使用线程 #

有三种使用线程的方法：

- 实现Runnable接口；
- 实现Callable接口；
- 继承Thread类；

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以理解为任务是通过线程驱动从而执行的。

## 实现Runnable接口

需要实现接口中的run()方法。

	public class MyRunnable implements Runnable {
	    @Override
	    public void run() {
	        // ...
	    }
	}

使用 Runnable 实例再创建一个 Thread 实例，然后调用 Thread 实例的 start() 方法来启动线程。
	
	public static void main(String[] args) {
	    MyRunnable instance = new MyRunnable();
	    Thread thread = new Thread(instance);
	    thread.start();
	}

## 实现Callable接口 ##

与 Runnable 相比，Callable 可以有返回值，返回值通过 FutureTask 进行封装。


	public class MyCallable implements Callable<Integer> {
	    public Integer call() {
	        return 123;
	    }
	}

	public static void main(String[] args) throws ExecutionException, InterruptedException {
	    MyCallable mc = new MyCallable();
	    FutureTask<Integer> ft = new FutureTask<>(mc);
	    Thread thread = new Thread(ft);
	    thread.start();
	    System.out.println(ft.get());
	}

## 继承Tread类

同样也是需要实现 run() 方法，因为 Thread 类也实现了 Runable 接口。

当调用 start() 方法启动一个线程时，虚拟机会将该线程放入就绪队列中等待被调度，当一个线程被调度时会执行该线程的 run() 方法。

	public class MyThread extends Thread {
	    public void run() {
	        // ...
	    }
	}
	
	public static void main(String[] args) {
	    MyThread mt = new MyThread();
	    mt.start();
	}

## 实现接口 VS 继承 Thread

实现接口会更好一些，因为：

- Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；
- 类可能只要求可执行就行，继承整个 Thread 类开销过大。

# 2.基础线程机制

## Executer

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种Executor：

- CachedThreadPool：根据需要创建线程的线程池，一个任务就需要一个线程，提交任务的速度 > 线程池中线程处理任务的速度就要不断创建新线程；
- FixedThreadPool：所有任务只能使用固定大小的线程池；
- SingleThreadExecutor：相当于大小为 1 的 FixedThreadPool。

		public static void main(String[] args) {
		    ExecutorService executorService = Executors.newCachedThreadPool();
		    for (int i = 0; i < 5; i++) {
		        executorService.execute(new MyRunnable());
		    }
		    executorService.shutdown();
		}

## Daemon

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于非守护线程。

在线程启动之前使用 setDaemon() 方法可以将一个线程设置为守护线程。

	public static void main(String[] args) {
	    Thread thread = new Thread(new MyRunnable());
	    thread.setDaemon(true);
	}

## sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

	public void run() {
	    try {
	        Thread.sleep(3000);
	    } catch (InterruptedException e) {
	        e.printStackTrace();
	    }
	}

## yield()

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

	public void run() {
	    Thread.yield();
	}

# 3.中断 #

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

## InterruptedException ##

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

	public class InterruptExample {
	
	    private static class MyThread1 extends Thread {
	        @Override
	        public void run() {
	            try {
	                Thread.sleep(2000);
	                System.out.println("Thread run");
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        }
	    }
	}
	
	public static void main(String[] args) throws InterruptedException {
	    Thread thread1 = new MyThread1();
	    thread1.start();
	    thread1.interrupt();
	    System.out.println("Main run");
	}

----------

	Main run
	java.lang.InterruptedException: sleep interrupted
	    at java.lang.Thread.sleep(Native Method)
	    at InterruptExample.lambda$main$0(InterruptExample.java:5)
	    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
	    at java.lang.Thread.run(Thread.java:745)

## interrupted()

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

	public class InterruptExample {
	
	    private static class MyThread2 extends Thread {
	        @Override
	        public void run() {
	            while (!interrupted()) {
	                // ..
	            }
	            System.out.println("Thread end");
	        }
	    }
	}
	
	public static void main(String[] args) throws InterruptedException {
	    Thread thread2 = new MyThread2();
	    thread2.start();
	    thread2.interrupt();
	}


----------

	Thread end


## Executor 的中断操作 ##

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

以下使用 Lambda 创建线程，相当于创建了一个匿名内部线程。

	public static void main(String[] args) {
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    executorService.execute(() -> {
	        try {
	            Thread.sleep(2000);
	            System.out.println("Thread run");
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	    });
	    executorService.shutdownNow();
	    System.out.println("Main run");
	}


----------

	Main run
	java.lang.InterruptedException: sleep interrupted
	    at java.lang.Thread.sleep(Native Method)
	    at ExecutorInterruptExample.lambda$main$0(ExecutorInterruptExample.java:9)
	    at ExecutorInterruptExample$$Lambda$1/1160460865.run(Unknown Source)
	    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	    at java.lang.Thread.run(Thread.java:745)

如果只想中断 Executor 中的一个线程，可以通过使用 submit() 方法来提交一个线程，它会返回一个 Future<?> 对象，通过调用该对象的 cancel(true) 方法就可以中断线程。

	Future<?> future = executorService.submit(() -> {
	    // ..
	});
	future.cancel(true);

# 4.互斥同步 

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

## synchronized

每个 Java 对象都有一个关联的 monitor，使用 synchronized 时 JVM 会根据使用环境找到对象的 monitor，根据 monitor 的状态进行加解锁的判断。如果成功加锁就成为该 monitor 的唯一持有者，monitor 在被释放前不能再被其他线程获取。 

同步代码块使用 monitorenter 和 monitorexit 这两个字节码指令获取和释放 monitor。这两个字节码指令都需要一个引用类型的参数指明要锁定和解锁的对象，对于同步普通方法，锁是当前实例对象；对于静态同步方法，锁是当前类的 Class 对象；对于同步方法块，锁是 synchronized 括号里的对象。

### 1. 同步一个代码块

	public void func() {
	    synchronized (this) {
	        // ...
	    }
	}

它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

	public class SynchronizedExample {
	
	    public void func1() {
	        synchronized (this) {
	            for (int i = 0; i < 10; i++) {
	                System.out.print(i + " ");
	            }
	        }
	    }
	}

----------

	public static void main(String[] args) {
	    SynchronizedExample e1 = new SynchronizedExample();
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    executorService.execute(() -> e1.func1());
	    executorService.execute(() -> e1.func1());
	}


----------

	0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9

对于以下代码，两个线程调用了不同对象的同步代码块，因此这两个线程就不需要同步。从输出结果可以看出，两个线程交叉执行。

	public static void main(String[] args) {
	    SynchronizedExample e1 = new SynchronizedExample();
	    SynchronizedExample e2 = new SynchronizedExample();
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    executorService.execute(() -> e1.func1());
	    executorService.execute(() -> e2.func1());
	}


----------

	0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9

### 2. 同步一个方法

	public synchronized void func () {
	    // ...
	}

它和同步代码块一样，作用于同一个对象。

### 3. 同步一个类

	public void func() {
	    synchronized (SynchronizedExample.class) {
	        // ...
	    }
	}

作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

	public class SynchronizedExample {
	
	    public void func2() {
	        synchronized (SynchronizedExample.class) {
	            for (int i = 0; i < 10; i++) {
	                System.out.print(i + " ");
	            }
	        }
	    }
	}


----------

	public static void main(String[] args) {
	    SynchronizedExample e1 = new SynchronizedExample();
	    SynchronizedExample e2 = new SynchronizedExample();
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    executorService.execute(() -> e1.func2());
	    executorService.execute(() -> e2.func2());
	}

----------

	0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9


----------

### 4. 同步一个静态方法

	public synchronized static void fun() {
	    // ...
	}

作用于整个类。

## synchronized的锁优化 ##

### 自旋锁 ###

互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

### 锁消除 ###

锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。

### 锁粗化 ###

如果虚拟机探测到由这样的一串零碎的操作都对同一个对象加锁，将会把加锁的范围扩展（粗化）到整个操作序列的外部。

### 轻量级锁 ###

轻量级锁是相对于传统的重量级锁而言，它使用 CAS 操作来避免重量级锁使用互斥量的开销。对于绝大部分的锁，在整个同步周期内都是不存在竞争的，因此也就不需要都使用互斥量进行同步，可以先采用 CAS 操作进行同步，如果 CAS 失败了再改用互斥量进行同步。

### 偏向锁 ###

偏向锁的思想是偏向于让第一个获取锁对象的线程，这个线程在之后获取该锁就不再需要进行同步操作，甚至连 CAS 操作也不再需要。

## synchronized不公平的原因 ##

所有收到锁请求的线程首先自旋，如果通过自旋也没有获取锁将被放入 ContentionList，该做法对于已经进入队列的线程不公平。 

为了防止 ContentionList 尾部的元素被大量线程进行 CAS 访问影响性能，Owner 线程会在释放锁时将 ContentionList 的部分线程移动到 EntryList 并指定某个线程为 OnDeck 线程，该行为叫做竞争切换，牺牲了公平性但提高了性能。

## ReentrantLock

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

在ReentrantLock中，调用lock()方法获取锁；调用unlock()方法释放锁。

	public class LockExample {
	
	    private Lock lock = new ReentrantLock();
	
	    public void func() {
	        lock.lock();
	        try {
	            for (int i = 0; i < 10; i++) {
	                System.out.print(i + " ");
	            }
	        } finally {
	            lock.unlock(); // 确保释放锁，从而避免发生死锁。
	        }
	    }
	}


----------

	public static void main(String[] args) {
	    LockExample lockExample = new LockExample();
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    executorService.execute(() -> lockExample.func());
	    executorService.execute(() -> lockExample.func());
	}


----------

	0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8

ReentrantLock的实现依赖于java同步器框架AQS。AQS使用一个整型的volatile变量（命名为state）来维护同步状态。

> AQS 队列同步器是用来构建锁或其他同步组件的基础框架，它使用一个 volatile int state 变量作为共享资源，如果线程获取资源失败，则进入同步队列等待；如果获取成功就执行临界区代码，释放资源时会通知同步队列中的等待线程。

公平锁在释放锁的最后写volatile变量state；在获取锁时首先读这个volatile变量。根据volatile的happens-before规则，释放锁的线程在写volatile变量之前可见的共享变量，在获取锁的线程读取同一个volatile变量后将立即变的对获取锁的线程可见。

- 公平锁和非公平锁释放时，最后都要写一个volatile变量state。
- 公平锁获取时，首先会去读这个volatile变量。
- 非公平锁获取时，首先会用CAS更新这个volatile变量,这个操作同时具有volatile读和volatile写的内存语义。

## 如何避免死锁 ##

使用多线程可以提高性能，但是对一些情况会出现线程不安全的问题，为了避免线程不安全问题一般我们是加锁，然后加锁就会出现死锁问题，一般我们Syncronize或ReentrantLock给代码加锁，Syncronize锁住代码块时JVM级别的，会自己解锁，但是使用ReentrantLock的时候一般是搭配try,catch代码块使用，在finally中释放锁。同时配合wait使用，防止一直占有锁。

## 比较

### 1. 锁的实现 ###

synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

### 2. 性能

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

### 3. 等待可中断

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

### 4. 公平锁

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

### 5. 锁绑定多个条件

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

## 使用选择

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。

## 读写锁 ##

ReentrantLock 是排他锁，同一时刻只允许一个线程访问，读写锁在同一时刻允许多个读线程访问，在写线程访问时，所有的读写线程均阻塞。读写锁维护了一个读锁和一个写锁，通过分离读写锁使并发性相比排他锁有了很大提升。 

读写锁依赖 AQS 来实现同步功能，读写状态就是其同步器的同步状态。读写锁的自定义同步器需要在同步状态，即一个 int 变量上维护多个读线程和一个写线程的状态。读写锁将变量切分成了两个部分，高 16 位表示读，低 16 位表示写。

# 五、线程之间的协作 #

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

## join() ##

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。

	public class JoinExample {
	
	    private class A extends Thread {
	        @Override
	        public void run() {
	            System.out.println("A");
	        }
	    }
	
	    private class B extends Thread {
	
	        private A a;
	
	        B(A a) {
	            this.a = a;
	        }
	
	        @Override
	        public void run() {
	            try {
	                a.join();
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	            System.out.println("B");
	        }
	    }
	
	    public void test() {
	        A a = new A();
	        B b = new B(a);
	        b.start();
	        a.start();
	    }
	}

----------

	A
	B


## wait() notify() notifyAll() ##

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

	public class WaitNotifyExample {
	
	    public synchronized void before() {
	        System.out.println("before");
	        notifyAll();
	    }
	
	    public synchronized void after() {
	        try {
	            wait();
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	        System.out.println("after");
	    }
	}

----------

	public static void main(String[] args) {
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    WaitNotifyExample example = new WaitNotifyExample();
	    executorService.execute(() -> example.after());
	    executorService.execute(() -> example.before());
	}

----------

	before
	after

wait() 和 sleep() 的区别

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

## await() signal() signalAll() ##

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。

	public class AwaitSignalExample {
	
	    private Lock lock = new ReentrantLock();
	    private Condition condition = lock.newCondition();
	
	    public void before() {
	        lock.lock();
	        try {
	            System.out.println("before");
	            condition.signalAll();
	        } finally {
	            lock.unlock();
	        }
	    }
	
	    public void after() {
	        lock.lock();
	        try {
	            condition.await();
	            System.out.println("after");
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        } finally {
	            lock.unlock();
	        }
	    }
	}


----------

	public static void main(String[] args) {
	    ExecutorService executorService = Executors.newCachedThreadPool();
	    AwaitSignalExample example = new AwaitSignalExample();
	    executorService.execute(() -> example.after());
	    executorService.execute(() -> example.before());
	}

----------

	before
	after

# 六、线程状态 #

一个线程只能处于一种状态，并且这里的线程状态特指 Java 虚拟机的线程状态，不能反映线程在特定操作系统下的状态。

共六个状态。

### 新建（NEW） ###

创建后尚未启动。

### 可运行（RUNABLE） ###

正在 Java 虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。

### 阻塞（BLOCKED） ###

请求获取 monitor lock 从而进入 synchronized 函数或者代码块，但是其它线程已经占用了该 monitor lock，所以出于阻塞状态。要结束该状态进入从而 RUNABLE 需要其他线程释放 monitor lock。

### 无限期等待（WAITING） ###

等待其它线程显式地唤醒。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取 monitor lock。而等待是主动的，通过调用 Object.wait() 等方法进入。

进入方法	|退出方法
-|-
没有设置 Timeout 参数的 Object.wait() |方法	Object.notify() / Object.notifyAll()
没有设置 Timeout 参数的 Thread.join() |方法	被调用的线程执行完毕
LockSupport.park() 方法	|LockSupport.unpark(Thread)

### 限期等待（TIMED_WAITING） ###

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

进入方法	|退出方法
-|-
Thread.sleep() 方法|	时间结束
设置了 Timeout 参数的 Object.wait() 方法	|时间结束 / Object.notify() / Object.notifyAll()
设置了 Timeout 参数的 Thread.join() 方法	|时间结束 / 被调用的线程执行完毕
LockSupport.parkNanos() 方法|	LockSupport.unpark(Thread)
LockSupport.parkUntil() 方法|	LockSupport.unpark(Thread)

### 死亡（TERMINATED） ###

可以是线程结束任务之后自己结束，或者产生了异常而结束。

# 七、Java 内存模型（JMM） #

Java 线程的通信由 JMM 控制，JMM 的主要目的是定义程序中各种变量的访问规则。变量包括实例字段、静态字段，但不包括局部变量与方法参数，因为它们是线程私有的，不存在多线程竞争。JMM 遵循一个基本原则：只要不改变程序执行结果，编译器和处理器怎么优化都行。

JMM 规定所有变量都存储在主内存，每条线程有自己的工作内存，工作内存中保存被该线程使用的变量的主内存副本，线程对变量的所有操作都必须在工作空间进行，不能直接读写主内存数据。不同线程间无法直接访问对方工作内存中的变量，线程通信必须经过主内存。

关于主内存与工作内存的交互，JMM 定义了 8 种原子操作。

- read：把一个变量的值从主内存传输到工作内存中
- load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
- use：把工作内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在 store 之后执行，把 store 得到的值放入主内存的变量中
- lock：作用于主内存的变量
- unlock

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/8b7ebbad-9604-4375-84e3-f412099d170c.png)

## 内存模型三大特性 ##

### 1. 原子性 ###

Java 内存模型保证了 read、load、use、assign、store、write、lock 和 unlock 操作具有原子性，例如对一个 int 类型的变量执行 assign 赋值操作，这个操作就是原子性的。但是 Java 内存模型允许虚拟机将没有被 volatile 修饰的 64 位数据（long，double）的读写操作划分为两次 32 位的操作来进行，即 load、store、read 和 write 操作可以不具备原子性。

### 2. 可见性 ###

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有三种实现可见性的方式：

- volatile
- synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
- final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

### 3. 有序性 ###

有序性是指：在本线程内观察，所有操作都是有序的。在一个线程观察另一个线程，所有操作都是无序的，无序是因为发生了指令重排序。在 Java 内存模型中，允许编译器和处理器对指令进行重排序，重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

也可以通过 synchronized 来保证有序性，它保证每个时刻只有一个线程执行同步代码，相当于是让线程顺序执行同步代码。

## volatile ##

- 原子性：对任意单个volatile变量的读/写具有原子性。即使是64位的long型和double型变量，只要它是volatile变量，对该变量的读写就将具有原子性。如果是多个volatile操作或类似于volatile++这种复合操作，这些操作整体上不具有原子性。
- 可见性：一个线程修改了共享变量的值，其它线程能够立即得知这个修改。
- 有序性：volatile 关键字通过添加内存屏障的方式来禁止指令重排，即重排序时不能把后面的指令放到内存屏障之前。

## as-if-serial ##

不管怎么重排序，单线程程序的执行结果不能改变，编译器和处理器必须遵循 as-if-serial 语义。 

为了遵循 as-if-serial，编译器和处理器不会对存在数据依赖关系的操作重排序，因为这种重排序会改变执行结果。但是如果操作之间不存在数据依赖关系，这些操作就可能被编译器和处理器重排序。 

as-if-serial 把单线程程序保护起来，给程序员一种幻觉：单线程程序是按程序的顺序执行的。

## happens-before（先发先行原则） ##

先行发生原则，JMM 定义的两项操作间的偏序关系，是判断数据是否存在竞争的重要手段。 

JMM 将 happens-before 要求禁止的重排序按是否会改变程序执行结果分为两类。对于会改变结果的重排序 JMM 要求编译器和处理器必须禁止，对于不会改变结果的重排序，JMM 不做要求。  

JMM 存在一些天然的 happens-before 关系，无需任何同步器协助就已经存在，一个操作无需控制就能先于另一个操作完成。如果两个操作的关系不在此列，并且无法从这些规则推导出来，它们就没有顺序性保障，虚拟机可以对它们随意进行重排序。

- 程序次序规则：一个线程内写在前面的操作先行发生于后面的。 
- 管程锁定规则： unlock 操作先行发生于后面对同一个锁的 lock 操作。 
- volatile 规则：对 volatile 变量的写操作先行发生于后面的读操作。 
- 线程启动规则：线程的 start 方法先行发生于线程的每个动作。 
- 线程终止规则：线程中所有操作先行发生于对线程的终止检测。 
- 对象终结规则：对象的初始化先行发生于 finalize 方法。 
- 传递性：如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C 。 

## as-if-serial 和 happens-before 有什么区别？ ##

as-if-serial 保证单线程程序的执行结果不变，happens-before 保证正确同步的多线程程序的执行结果不变。


# 八、 线程安全 #

多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

## 不可变 ##

不可变（Immutable）的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象成为不可变，来满足线程安全。

不可变的类型：

- final 关键字修饰的基本数据类型
- String
- 枚举类型
- Number 部分子类，如 Long 和 Double 等数值包装类型，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInteger 和 AtomicLong 则是可变的。

## 互斥同步 ##

synchronized 和 ReentrantLock。

## 非阻塞同步 ##

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施（不断地重试，重复读-比较-写，直到成功为止）。实现：先读取版本号，然后再操作，比较此时和读入的版本号，如果相同则更新数据，负责（重复这个过程）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。



### 1. CAS 

CAS是一种更新的原子操作，比较当前值与传入值是否一样，一样则更新，否则则失败。

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

### 2. AtomicInteger 

J.U.C 包里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。

以下代码是 getAndAddInt() 源码，var1 指示对象内存地址，var2 指示该字段相对对象内存地址的偏移，var4 指示操作需要加的数值，这里为 1。通过 getIntVolatile(var1, var2) 得到旧的预期值，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 var5，那么就更新内存地址为 var1+var2 的变量为 var5+var4。

可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。

	public final int getAndAddInt(Object var1, long var2, int var4) {
	    int var5;
	    do {
	        var5 = this.getIntVolatile(var1, var2);
	    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
	
	    return var5;
	}

### 3. ABA 

如果一个变量初次读取的时候是 A 值，它的值被改成了 B，后来又被改回为 A，那 CAS 操作就会误认为它从来没有被改变过。

J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它可以通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步可能会比原子类更高效。

## 无同步方案 ##

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

### 1. 栈封闭 ###

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

### 2. 线程本地存储（Thread Local Storage） ###

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消费队列的架构模式（如“生产者-消费者”模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的“一个请求对应一个服务器线程”（Thread-per-Request）的处理方式，这种处理方式的广泛应用使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能。

对于以下代码，thread1 中设置 threadLocal 为 1，而 thread2 设置 threadLocal 为 2。过了一段时间之后，thread1 读取 threadLocal 依然是 1，不受 thread2 的影响。

	public class ThreadLocalExample {
	    public static void main(String[] args) {
	        ThreadLocal threadLocal = new ThreadLocal();
	        Thread thread1 = new Thread(() -> {
	            threadLocal.set(1);
	            try {
	                Thread.sleep(1000);
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	            System.out.println(threadLocal.get());
	            threadLocal.remove();
	        });
	        Thread thread2 = new Thread(() -> {
	            threadLocal.set(2);
	            threadLocal.remove();
	        });
	        thread1.start();
	        thread2.start();
	    }
	}

每个 Thread 都有一个 ThreadLocal.ThreadLocalMap 对象。

当调用一个 ThreadLocal 的 set(T value) 方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 ThreadLocal->value 键值对插入到该 Map 中。

get() 方法类似。



