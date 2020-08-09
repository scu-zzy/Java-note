## 生产者消费者 ##

### wait()和notify()方法的实现 ###

	public class Test1 {
	    private static Integer count = 0;
	    private static final Integer FULL = 10;
	    private static String LOCK = "lock";
	
	    public static void main(String[] args) {
	        Test1 test1 = new Test1();
	        new Thread(test1.new Producer()).start();
	        new Thread(test1.new Consumer()).start();
	        new Thread(test1.new Producer()).start();
	        new Thread(test1.new Consumer()).start();
	    }
	
	    class Producer implements Runnable {
	        @Override
	        public void run() {
	            while (true){
	                synchronized (LOCK) {
	                    while (count == FULL) {
	                        try {
	                            LOCK.wait();
	                        } catch (Exception e) {
	                            e.printStackTrace();
	                        }
	                    }
	                    count++;
	                    System.out.println(Thread.currentThread().getName() + "生产者生产，目前总共有" + count);
	                    LOCK.notifyAll();
	                }
	            }
	        }
	    }
	
	    class Consumer implements Runnable {
	        @Override
	        public void run() {
	            while (true){
	                synchronized (LOCK) {
	                    while (count == 0) {
	                        try {
	                            LOCK.wait();
	                        } catch (Exception e) {
	                            e.printStackTrace();
	                        }
	                    }
	                    count--;
	                    System.out.println(Thread.currentThread().getName() + "消费者消费，目前总共有" + count);
	                    LOCK.notifyAll();
	                }
	            }
	        }
	    }
	}

### 可重入锁ReentrantLock的实现（await和signal） ###

java.util.concurrent.lock 中的 Lock 框架是锁定的一个抽象，通过对lock的lock()方法和unlock()方法实现了对锁的显示控制，而synchronize()则是对锁的隐性控制。 
可重入锁，也叫做递归锁，指的是同一线程 外层函数获得锁之后 ，内层递归函数仍然有获取该锁的代码，但不受影响，简单来说，该锁维护这一个与获取锁相关的计数器，如果拥有锁的某个线程再次得到锁，那么获取计数器就加1，函数调用结束计数器就减1，然后锁需要被释放两次才能获得真正释放。已经获取锁的线程进入其他需要相同锁的同步代码块不会被阻塞。

	import java.util.concurrent.locks.Condition;
	import java.util.concurrent.locks.Lock;
	import java.util.concurrent.locks.ReentrantLock;
	
	public class Test2 {
	    private static Integer count = 0;
	    private static final Integer FULL = 10;
	    //创建一个锁对象
	    private Lock lock = new ReentrantLock();
	    //创建两个条件变量，一个为缓冲区非满，一个为缓冲区非空
	    private final Condition notFull = lock.newCondition();
	    private final Condition notEmpty = lock.newCondition();
	
	    public static void main(String[] args) {
	        Test2 test2 = new Test2();
	        new Thread(test2.new Producer()).start();
	        new Thread(test2.new Consumer()).start();
	        new Thread(test2.new Producer()).start();
	        new Thread(test2.new Consumer()).start();
	        new Thread(test2.new Producer()).start();
	        new Thread(test2.new Consumer()).start();
	        new Thread(test2.new Producer()).start();
	        new Thread(test2.new Consumer()).start();
	    }
	    class Producer implements Runnable {
	        @Override
	        public void run() {
	            while (true){
	                //获取锁
	                lock.lock();
	                try {
	                    while (count == FULL) {
	                        try {
	                            notFull.await();
	                        } catch (InterruptedException e) {
	                            e.printStackTrace();
	                        }
	                    }
	                    count++;
	                    System.out.println(Thread.currentThread().getName()
	                            + "生产者生产，目前总共有" + count);
	                    //唤醒消费者
	                    notEmpty.signal();
	                } finally {
	                    //释放锁
	                    lock.unlock();
	                }
	            }
	        }
	    }
	
	    class Consumer implements Runnable {
	        @Override
	        public void run() {
	            while (true){
	                lock.lock();
	                try {
	                    while (count == 0) {
	                        try {
	                            notEmpty.await();
	                        } catch (Exception e) {
	
	                            e.printStackTrace();
	                        }
	                    }
	                    count--;
	                    System.out.println(Thread.currentThread().getName()
	                            + "消费者消费，目前总共有" + count);
	                    notFull.signal();
	                } finally {
	                    lock.unlock();
	                }
	            }
	        }
	    }
	}

### 信号量 ###

	import java.util.concurrent.Semaphore;
	
	public class Test4 {
	    private static Integer count = 0;
	    //创建三个信号量
	    final Semaphore notFull = new Semaphore(10);
	    final Semaphore notEmpty = new Semaphore(0);
	    final Semaphore mutex = new Semaphore(1);
	
	    public static void main(String[] args) {
	        Test4 test4 = new Test4();
	        new Thread(test4.new Producer()).start();
	        new Thread(test4.new Consumer()).start();
	        new Thread(test4.new Producer()).start();
	        new Thread(test4.new Consumer()).start();
	    }
	    class Producer implements Runnable {
	        @Override
	        public void run() {
	            while (true){
	                try {
	                    notFull.acquire();
	                    mutex.acquire();
	                    count++;
	                    System.out.println(Thread.currentThread().getName()
	                            + "生产者生产，目前总共有" + count);
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                } finally {
	                    mutex.release();
	                    notEmpty.release();
	                }
	            }
	        }
	    }
	
	    class Consumer implements Runnable {
	
	        @Override
	        public void run() {
	            while (true){
	                try {
	                    notEmpty.acquire();
	                    mutex.acquire();
	                    count--;
	                    System.out.println(Thread.currentThread().getName()
	                            + "消费者消费，目前总共有" + count);
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                } finally {
	                    mutex.release();
	                    notFull.release();
	                }
	            }
	        }
	    }
	}

### 阻塞队列 ###

有点问题

下面来看由阻塞队列实现的生产者消费者模型,这里我们使用take()和put()方法，这里生产者和生产者，消费者和消费者之间不存在同步，所以会出现连续生成和连续消费的现象

	import java.util.concurrent.ArrayBlockingQueue;
	import java.util.concurrent.BlockingQueue;
	
	public class Test3 {
	
	    //创建一个阻塞队列
	    final BlockingQueue<Integer> blockingQueue = new ArrayBlockingQueue<>(10);
	
	    public static void main(String[] args) {
	        Test3 test3 = new Test3();
	        new Thread(test3.new Producer()).start();
	        new Thread(test3.new Consumer()).start();
	        new Thread(test3.new Producer()).start();
	        new Thread(test3.new Consumer()).start();
	    }
	    class Producer implements Runnable {
	        @Override
	        public void run() {
	            while (true){
	                try {
	                    blockingQueue.put(1);
	                    System.out.println(Thread.currentThread().getName()
	                            + "生产者生产，目前总共有" + blockingQueue.size());
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	
	    class Consumer implements Runnable {
	
	        @Override
	        public void run() {
	            while (true){
	                try {
	                    blockingQueue.take();
	                    System.out.println(Thread.currentThread().getName()
	                            + "消费者消费，目前总共有" + blockingQueue.size());
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	}

## 读者写者问题 ##

信号量

	import java.util.concurrent.Semaphore;
	
	public class Test5 {
	    private static Integer count = 0;//记录读者的数量
	    //创建信号量
	    final Semaphore countMutex = new Semaphore(1);//对count加锁
	    final Semaphore dataMutex = new Semaphore(1);//对读者和写者加锁
	
	    public static void main(String[] args) {
	        Test5 test5 = new Test5();
	        new Thread(test5.new Writer()).start();
	        new Thread(test5.new Reader()).start();
	    }
	    class Reader implements Runnable {
	        @Override
	        public void run() {
	            while (true){
	                try {
	                    countMutex.acquire();
	                    count++;
	                    if(count == 1) dataMutex.acquire();//第一个读者来时，加锁，阻止写者访问
	                    countMutex.release();
	                    System.out.println("read");
	                    countMutex.acquire();
	                    count--;
	                    if(count == 0) dataMutex.release();//最后一个读者走时，释放锁
	                    countMutex.release();
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	
	    class Writer implements Runnable {
	
	        @Override
	        public void run() {
	            while (true){
	                try {
	                    dataMutex.acquire();
	                    System.out.println("Write");
	                    dataMutex.release();
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	}

## 哲学家进餐问题 ##

每个哲学家必须确定自己左右手的筷子都可用的时候，才能同时拿起两只筷子进餐，吃完之后同时放下两只筷子。

	public class Test6 {
	    /*每个哲学家相当于一个线程*/
	    static class Philosopher extends Thread{
	        private String name;
	        private Fork fork;
	        public Philosopher(String name,Fork fork){
	            super(name);
	            this.name=name;
	            this.fork=fork;
	        }
	
	        public void run(){
	            while(true){
	                thinking();
	                fork.takeFork();
	                eating();
	                fork.putFork();
	            }
	
	        }
	
	        public void eating(){
	            System.out.println("I am Eating:"+name);
	            try {
	                sleep(1000);//模拟吃饭，占用一段时间资源
	            } catch (InterruptedException e) {
	                // TODO Auto-generated catch block
	                e.printStackTrace();
	            }
	        }
	
	
	        public void thinking(){
	            System.out.println("I am Thinking:"+name);
	            try {
	                sleep(1000);//模拟思考
	            } catch (InterruptedException e) {
	                // TODO Auto-generated catch block
	                e.printStackTrace();
	            }
	        }
	    }
	
	    static class Fork{
	        /*5只筷子，初始为都未被用*/
	        private boolean[] used={false,false,false,false,false,false};
	
	        /*只有当左右手的筷子都未被使用时，才允许获取筷子，且必须同时获取左右手筷子*/
	        public synchronized void takeFork(){
	            String name = Thread.currentThread().getName();
	            int i = Integer.parseInt(name);
	            while(used[i]||used[(i+1)%5]){
	                try {
	                    wait();//如果左右手有一只正被使用，等待
	                } catch (InterruptedException e) {
	                    // TODO Auto-generated catch block
	                    e.printStackTrace();
	                }
	            }
	            used[i ]= true;
	            used[(i+1)%5]=true;
	        }
	
	        /*必须同时释放左右手的筷子*/
	        public synchronized void putFork(){
	            String name = Thread.currentThread().getName();
	            int i = Integer.parseInt(name);
	
	            used[i ]= false;
	            used[(i+1)%5]=false;
	            notifyAll();//唤醒其他线程
	        }
	    }
	
	    public static void main(String []args){
	        Fork fork = new Fork();
	        new Philosopher("0",fork).start();
	        new Philosopher("1",fork).start();
	        new Philosopher("2",fork).start();
	        new Philosopher("3",fork).start();
	        new Philosopher("4",fork).start();
	    }
	}

## 1114. 按序打印 ##

我们提供了一个类：

	public class Foo {
	  public void one() { print("one"); }
	  public void two() { print("two"); }
	  public void three() { print("three"); }
	}

三个不同的线程将会共用一个 Foo 实例。


- 	线程 A 将会调用 one() 方法
- 	线程 B 将会调用 two() 方法
- 	线程 C 将会调用 three() 方法


请设计修改程序，以确保 two() 方法在 one() 方法之后被执行，three() 方法在 two() 方法之后被执行。

示例 1:

	输入: [1,2,3]
	输出: "onetwothree"
	解释: 
	有三个线程会被异步启动。
	输入 [1,2,3] 表示线程 A 将会调用 one() 方法，线程 B 将会调用 two() 方法，线程 C 将会调用 three() 方法。
	正确的输出是 "onetwothree"。

示例 2:

	输入: [1,3,2]
	输出: "onetwothree"
	解释: 
	输入 [1,3,2] 表示线程 A 将会调用 one() 方法，线程 B 将会调用 three() 方法，线程 C 将会调用 two() 方法。
	正确的输出是 "onetwothree"。

注意:
尽管输入中的数字似乎暗示了顺序，但是我们并不保证线程在操作系统中的调度顺序。
你看到的输入格式主要是为了确保测试的全面性。

思路：

方法一：原子类

- 首先初始化共享变量 firstJobDone 和 secondJobDone，初始值表示所有方法未执行。


- 方法 first() 没有依赖关系，可以直接执行。在方法最后更新变量 firstJobDone 表示该方法执行完成。


- 方法 second() 中，检查 firstJobDone 的状态。如果未更新则进入等待状态，否则执行方法 second()。在方法末尾，更新变量 secondJobDone 表示方法 second() 执行完成。


- 方法 third() 中，检查 secondJobDone 的状态。与方法 second() 类似，执行 third() 之前，需要先等待 secondJobDone 的状态。

		class Foo {
		
		  private AtomicInteger firstJobDone = new AtomicInteger(0);
		  private AtomicInteger secondJobDone = new AtomicInteger(0);
		
		  public Foo() {}
		
		  public void first(Runnable printFirst) throws InterruptedException {
		    // printFirst.run() outputs "first".
		    printFirst.run();
		    // mark the first job as done, by increasing its count.
		    firstJobDone.incrementAndGet();
		  }
		
		  public void second(Runnable printSecond) throws InterruptedException {
		    while (firstJobDone.get() != 1) {
		      // waiting for the first job to be done.
		    }
		    // printSecond.run() outputs "second".
		    printSecond.run();
		    // mark the second as done, by increasing its count.
		    secondJobDone.incrementAndGet();
		  }
		
		  public void third(Runnable printThird) throws InterruptedException {
		    while (secondJobDone.get() != 1) {
		      // waiting for the second job to be done.
		    }
		    // printThird.run() outputs "third".
		    printThird.run();
		  }
		}

方法二：锁加wait

	class Foo {
	    
	    private boolean firstFinished;
	    private boolean secondFinished;
	    private Object lock = new Object();
	
	    public Foo() {
	        
	    }
	
	    public void first(Runnable printFirst) throws InterruptedException {
	        
	        synchronized (lock) {
	            // printFirst.run() outputs "first". Do not change or remove this line.
	            printFirst.run();
	            firstFinished = true;
	            lock.notifyAll(); 
	        }
	    }
	
	    public void second(Runnable printSecond) throws InterruptedException {
	        
	        synchronized (lock) {
	            while (!firstFinished) {
	                lock.wait();
	            }
	        
	            // printSecond.run() outputs "second". Do not change or remove this line.
	            printSecond.run();
	            secondFinished = true;
	            lock.notifyAll();
	        }
	    }
	
	    public void third(Runnable printThird) throws InterruptedException {
	        
	        synchronized (lock) {
	           while (!secondFinished) {
	                lock.wait();
	            }
	
	            // printThird.run() outputs "third". Do not change or remove this line.
	            printThird.run();
	        } 
	    }
	}


## 1115. 交替打印FooBar ##

我们提供一个类：

	class FooBar {
	  public void foo() {
	    for (int i = 0; i < n; i++) {
	      print("foo");
	    }
	  }
	
	  public void bar() {
	    for (int i = 0; i < n; i++) {
	      print("bar");
	    }
	  }
	}

两个不同的线程将会共用一个 FooBar 实例。其中一个线程将会调用 foo() 方法，另一个线程将会调用 bar() 方法。

请设计修改程序，以确保 "foobar" 被输出 n 次。

1、信号量

	class FooBar {
	    private int n;
	
	    public FooBar(int n) {
	        this.n = n;
	    }
	
	    Semaphore foo = new Semaphore(1);
	    Semaphore bar = new Semaphore(0);
	
	    public void foo(Runnable printFoo) throws InterruptedException {
	        for (int i = 0; i < n; i++) {
	            foo.acquire();
	            printFoo.run();
	            bar.release();
	        }
	    }
	
	    public void bar(Runnable printBar) throws InterruptedException {
	        for (int i = 0; i < n; i++) {
	            bar.acquire();
	            printBar.run();
	            foo.release();
	        }
	    }
	}

2、volite变量控制

	class FooBar {
	    private int n;
	
	    public FooBar(int n) {
	        this.n = n;
	    }
	
	    volatile boolean permitFoo = true;
	
	    public void foo(Runnable printFoo) throws InterruptedException {     
	        for (int i = 0; i < n; ) {
	            if(permitFoo) {
	        		printFoo.run();
	            	i++;
	            	permitFoo = false;
	            }
	        }
	    }
	
	    public void bar(Runnable printBar) throws InterruptedException {       
	        for (int i = 0; i < n; ) {
	            if(!permitFoo) {
		        	printBar.run();
		        	i++;
		        	permitFoo = true;
	            }
	        }
	    }
	}

3、锁+wait

	class FooBar {
	    private int n;
	    private int count = 0;
	    
	    public FooBar(int n) {
	        this.n = n;
	    }
	
	    public synchronized void foo(Runnable printFoo) throws InterruptedException {
	        
	        for (int i = 0; i < n; i++) {
	            while(count == 1) this.wait();
	        	// printFoo.run() outputs "foo". Do not change or remove this line.
	        	printFoo.run();
	            count++;
	            this.notifyAll();
	        }
	    }
	
	    public synchronized void bar(Runnable printBar) throws InterruptedException {
	        
	        for (int i = 0; i < n; i++) {
	            while(count == 0) this.wait();            
	            // printBar.run() outputs "bar". Do not change or remove this line.
	        	printBar.run();
	            count--;
	            this.notifyAll();
	        }
	    }
	}