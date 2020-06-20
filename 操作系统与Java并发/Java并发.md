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

## 实现Callab接口 ##

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

