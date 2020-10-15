---
title: 'Java多线程(二) - 线程池'
date: 2020-09-02 15:28:02
tags: 
- code 
- 多线程
- Java
category: Java小技巧
---

#### 线程池的目的
1. 降低系统资源消耗，通过重用已存在的线程，降低线程创建和销毁造成的消耗
1. 提高系统响应速度，当有任务到达时，通过复用已存在的线程，无需等待新线程的创建便能立即执行
1. 方便线程并发数的管控。因为线程若是无限制的创建，可能会导致内存占用过多而产生[OOM](https://blog.csdn.net/xlgen157387/article/details/78298840 "OOM")，并且会造成cpu过度切换（cpu切换线程是有时间成本的（需要保持当前执行线程的现场，并恢复要执行线程的现场））
1. 提供更强大的功能，延时定时线程池

#### Executors提供的四种线程池
**- newSingleThreadExecutor**
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
	
**通俗**：创建只有一个线程的线程池，且线程的存活时间是无限的；当该线程正繁忙时，对于新任务会进入阻塞队列中(无界的阻塞队列)
	
**适用**：一个任务一个任务执行的场景
``` Java 
        ExecutorService pool = Executors.newSingleThreadExecutor();
        //函数式接口重写runnable方法
        pool.execute(() -> out.println(Thread.currentThread().getName()));
```
**- newFixedThreadPool**
创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
	
**通俗**：创建可容纳固定数量线程的池子，每个线程的存活时间是无限的，当池子满了就不在添加线程了；如果池中的所有线程均在繁忙状态，对于新任务会进入阻塞队列中(无界的阻塞队列)
	
**适用**：执行长期的任务，性能好很多
``` Java 
		/**
		* 1.创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小<br>
		* 2.线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程<br>
		* 3.因为线程池大小为3，每个任务输出index后sleep 2秒，所以每两秒打印3个数字，和线程名称<br>
		*/
		public static void fixTheadPoolTest() {
			ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
			for (int i = 0; i < 10; i++) {
				final int ii = i;
				fixedThreadPool.execute(() -> {
					out.println("线程名称：" + Thread.currentThread().getName() + "，执行" + ii);
					try {
						Thread.sleep(2000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				});
			}
		}
		------output-------
		线程名称：pool-1-thread-3，执行2
		线程名称：pool-1-thread-1，执行0
		线程名称：pool-1-thread-2，执行3
		线程名称：pool-1-thread-3，执行4
		线程名称：pool-1-thread-1，执行5
		线程名称：pool-1-thread-2，执行6
		线程名称：pool-1-thread-3，执行7
		线程名称：pool-1-thread-1，执行8
		线程名称：pool-1-thread-3，执行9

```
**- newScheduledThreadPool**
创建一个可定期或者延时执行任务的定长线程池，支持定时及周期性任务执行。 
	
**通俗**：创建一个固定大小的线程池，线程池内线程存活时间无限制，线程池可以支持定时及周期性任务执行，如果所有线程均处于繁忙状态，对于新任务会进入DelayedWorkQueue队列中，这是一种按照超时时间排序的队列结构
	
**适用**：周期性执行任务的场景
``` Java 
		/**
		 * 创建一个定长线程池，支持定时及周期性任务执行。延迟执行
		 */
		public static void sceduleThreadPool() {
			ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
			Runnable r1 = () -> out.println("线程名称：" + Thread.currentThread().getName() + "，执行:3秒后执行");
			scheduledThreadPool.schedule(r1, 3, TimeUnit.SECONDS);
			Runnable r2 = () -> out.println("线程名称：" + Thread.currentThread().getName() + "，执行:延迟2秒后每3秒执行一次");
			scheduledThreadPool.scheduleAtFixedRate(r2, 2, 3, TimeUnit.SECONDS);
			Runnable r3 = () -> out.println("线程名称：" + Thread.currentThread().getName() + "，执行:普通任务");
			for (int i = 0; i < 5; i++) {
				scheduledThreadPool.execute(r3);
			}
		}
		----output------
		线程名称：pool-1-thread-1，执行:普通任务
		线程名称：pool-1-thread-5，执行:普通任务
		线程名称：pool-1-thread-4，执行:普通任务
		线程名称：pool-1-thread-3，执行:普通任务
		线程名称：pool-1-thread-2，执行:普通任务
		线程名称：pool-1-thread-1，执行:延迟2秒后每3秒执行一次
		线程名称：pool-1-thread-5，执行:3秒后执行
		线程名称：pool-1-thread-4，执行:延迟2秒后每3秒执行一次
		线程名称：pool-1-thread-4，执行:延迟2秒后每3秒执行一次
		线程名称：pool-1-thread-4，执行:延迟2秒后每3秒执行一次
		线程名称：pool-1-thread-4，执行:延迟2秒后每3秒执行一次

```
**- newCachedThreadPool**
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
	
**通俗**：当有新任务到来，则插入到SynchronousQueue中，由于SynchronousQueue是同步队列，因此会在池中寻找可用线程来执行，若有可以线程则执行，若没有可用线程则创建一个线程来执行该任务；若池中线程空闲时间超过指定大小，则该线程会被销毁。
	
**适用**：执行很多短期异步的小程序或者负载较轻的服务器
``` Java 
		/**
		 * 1.创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程<br>
		 * 2.当任务数增加时，此线程池又可以智能的添加新线程来处理任务<br>
		 * 3.此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小<br>
		 * 
		 */
		public static void cacheThreadPool() {
			ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
			for (int i = 1; i <= 10; i++) {
				final int ii = i;
				try {
					Thread.sleep(ii * 1);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				cachedThreadPool.execute(()->out.println("线程名称：" + Thread.currentThread().getName() + "，执行" + ii));
			}
		}
		-----output------
		线程名称：pool-1-thread-1，执行1
		线程名称：pool-1-thread-1，执行2
		线程名称：pool-1-thread-1，执行3
		线程名称：pool-1-thread-1，执行4
		线程名称：pool-1-thread-1，执行5
		线程名称：pool-1-thread-1，执行6
		线程名称：pool-1-thread-1，执行7
		线程名称：pool-1-thread-1，执行8
		线程名称：pool-1-thread-1，执行9
		线程名称：pool-1-thread-1，执行10
```
#### [线程池的主要参数](https://blog.csdn.net/ye17186/article/details/89467919 "线程池的主要参数")
``` Java 
	public ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue) {
		this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
			 Executors.defaultThreadFactory(), defaultHandler);
	}
```
1. corePoolSize 线程池核心线程大小

	线程池中会维护一个最小的线程数量，即使这些线程处理空闲状态，他们也不会 被销毁，除非设置了allowCoreThreadTimeOut。这里的最小线程数量即是corePoolSize。

1. maximumPoolSize 线程池最大线程数量

	一个任务被提交到线程池后，首先会缓存到工作队列（后面会介绍）中，如果工作队列满了，则会创建一个新线程，然后从工作队列中的取出一个任务交由新线程来处理，而将刚提交的任务放入工作队列。线程池不会无限制的去创建新线程，它会有一个最大线程数量的限制，这个数量即由maximumPoolSize来指定。

1. keepAliveTime 空闲线程存活时间

	一个线程如果处于空闲状态，并且当前的线程数量大于corePoolSize，那么在指定时间后，这个空闲线程会被销毁，这里的指定时间由keepAliveTime来设定

1. unit 空间线程存活时间单位

	keepAliveTime的计量单位

1. workQueue 工作队列

	新任务被提交后，会先进入到此工作队列中，任务调度时再从队列中取出任务。jdk中提供了四种工作队列：

	①ArrayBlockingQueue

	基于数组的有界阻塞队列，按FIFO排序。新任务进来后，会放到该队列的队尾，有界的数组可以防止资源耗尽问题。当线程池中线程数量达到corePoolSize后，再有新任务进来，则会将任务放入该队列的队尾，等待被调度。如果队列已经是满的，则创建一个新线程，如果线程数量已经达到maxPoolSize，则会执行拒绝策略。

	②LinkedBlockingQueue

	基于链表的无界阻塞队列（其实最大容量为Integer.MAX），按照FIFO排序。由于该队列的近似无界性，当线程池中线程数量达到corePoolSize后，再有新任务进来，会一直存入该队列，而不会去创建新线程直到maxPoolSize，因此使用该工作队列时，参数maxPoolSize其实是不起作用的。

	③SynchronousQueue

	一个不缓存任务的阻塞队列，生产者放入一个任务必须等到消费者取出这个任务。也就是说新任务进来时，不会缓存，而是直接被调度执行该任务，如果没有可用线程，则创建新线程，如果线程数量达到maxPoolSize，则执行拒绝策略。

	④PriorityBlockingQueue

	具有优先级的无界阻塞队列，优先级通过参数Comparator实现。

1. threadFactory 线程工厂

	创建一个新线程时使用的工厂，可以用来设定线程名、是否为daemon线程等等

1. handler 拒绝策略

	当工作队列中的任务已到达最大限制，并且线程池中的线程数量也达到最大限制，这时如果有新任务提交进来，该如何处理呢。这里的拒绝策略，就是解决这个问题的，jdk中提供了4中拒绝策略：

	①CallerRunsPolicy

	该策略下，在调用者线程中直接执行被拒绝任务的run方法，除非线程池已经shutdown，则直接抛弃任务。

	②AbortPolicy

	该策略下，直接丢弃任务，并抛出RejectedExecutionException异常。
	
	③DiscardPolicy

	该策略下，直接丢弃任务，什么都不做。
	
	④DiscardOldestPolicy

	该策略下，抛弃进入队列最早的那个任务，然后尝试把这次拒绝的任务放入队列

#### 线程池流程

ThreadPoolExecutor执行execute()分4种情况

a、若当前运行的线程少于corePoolSize,则创建新线程来执行任务(执行这一步需要获取全局锁)
b、若运行的线程多于或等于corePoolSize,则将任务加入BlockingQueue
c、若无法将任务加入BlockingQueue,则创建新的线程来处理任务(执行这一步需要获取全局锁)
d、若创建新线程将使当前运行的线程超出maximumPoolSize,任务将被拒绝,并调用RejectedExecutionHandler.rejectedExecution()

采取上述思路,是为了在执行execute()时,尽可能避免获取全局锁
在ThreadPoolExecutor完成预热之后（当前运行的线程数大于等于corePoolSize),几乎所有的execute()方法调用都是执行步骤b,而步骤b不需要获取全局锁

**源码分析**：上面的流程分析让我们很直观的了解的线程池的工作原理，让我们再通过源代码来看看是如何实现的。线程池执行任务的方法如下：

``` Java 
	public void execute(Runnable command) {
		if (command == null)
			throw new NullPointerException();
		//如果线程数小于基本线程数，则创建线程并执行当前任务
		if (poolSize >= corePoolSize || !addIfUnderCorePoolSize(command)) {
			//如线程数大于等于基本线程数或线程创建失败，则将当前任务放到工作队列中。
			if (runState == RUNNING && workQueue.offer(command)) {
				if (runState != RUNNING || poolSize == 0)
					ensureQueuedTaskHandled(command);
			}
			//如果线程池不处于运行中或任务无法放入队列，并且当前线程数量小于最大允许的线程数量，则创建一个线程执行任务。
			else if (!addIfUnderMaximumPoolSize(command))
				//抛出RejectedExecutionException异常
				reject(command); // is shutdown or saturated
		}
	}
```

#### 如何配置线程池
**CPU密集型任务**
尽量使用较小的线程池，一般为CPU核心数+1。 因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，会造成CPU过度切换。

**IO密集型任务**
可以使用稍大的线程池，一般为2*CPU核心数。 IO密集型任务CPU使用率并不高，因此可以让CPU在等待IO的时候有其他线程去处理别的任务，充分利用CPU时间。

**混合型任务**
可以将任务分成IO密集型和CPU密集型任务，然后分别用不同的线程池去处理。 只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效。
因为如果划分之后两个任务执行时间有数据级的差距，那么拆分没有意义。
因为先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失。

#### LinkedBlockingQueue和ArrayBlockingQueue

　　一般如果线程池任务队列采用LinkedBlockingQueue队列的话，那么不会拒绝任何任务（因为队列大小没有限制），这种情况下，ThreadPoolExecutor最多仅会按照最小线程数来创建线程，也就是说线程池大小被忽略了。如果线程池任务队列采用

ArrayBlockingQueue队列的话，那么ThreadPoolExecutor将会采取一个非常负责的算法，比如假定线程池的最小线程数为4，最大为8所用的ArrayBlockingQueue最大为10。随着任务到达并被放到队列中，线程池中最多运行4个线程（即最小线程数）。

即使队列完全填满，也就是说有10个处于等待状态的任务，ThreadPoolExecutor也只会利用4个线程。如果队列已满，而又有新任务进来，此时才会启动一个新线程，这里不会因为队列已满而拒接该任务，相反会启动一个新线程。新线程会运行队列中的

第一个任务，为新来的任务腾出空间。这个算法背后的理念是：该池大部分时间仅使用核心线程（4个），即使有适量的任务在队列中等待运行。这时线程池就可以用作节流阀。如果挤压的请求变得非常多，这时该池就会尝试运行更多的线程来清理；

这时第二个节流阀—最大线程数就起作用了。

#### execute()和submit()方法

1、execute()，执行一个任务，没有返回值。
2、submit()，提交一个线程任务，有返回值。

**submit(Callable<T> task)**能获取到它的返回值，通过future.get()获取（阻塞直到任务执行完）。一般使用FutureTask+Callable配合使用（IntentService中有体现）。

**submit(Runnable task, T result)**能通过传入的载体result间接获得线程的返回值。
submit(Runnable task)则是没有返回值的，就算获取它的返回值也是null。

Future.get方法会使取结果的线程进入阻塞状态，直到线程执行完成之后，唤醒取结果的线程，然后返回结果。

#### 生产者和消费者模式
[**什么是生产者消费者模式**](http://ifeve.com/producers-and-consumers-mode/ "什么是生产者消费者模式")
生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。

线程池就是一个高明的消费者模式，新的线程“生产”的时候。如果没有到线程池的最大线程数量或者最内存，那么就直接创建一个线程并且把他“消费”掉，如果多了就扔到阻塞队列里。

#### 线程池演示

实例化一个runnable或者callable的类，然后直接传进线程池，不需要new一个thread然后start，或者new一个FutureTask了。

用runnable实现多线程（用的也算是pool.submit 但是没有返回值）
``` Java
	/**
	 *
	 * 线程池
	 * 跟数据库连接池类似
	 * 避免了线程的创建和销毁造成的额外开销
	 *
	 * java.util.concurrent
	 *
	 * Executor    负责现成的使用和调度的根接口
	 *    |--ExecutorService    线程池的主要接口
	 *          |--ThreadPoolExecutor    线程池的实现类
	 *          |--ScheduledExecutorService    接口，负责线程的调度
	 *              |--ScheduledThreadPoolExecutor    (extends ThreadPoolExecutor implements ScheduledExecutorService)
	 *
	 * Executors工具类
	 * 提供了创建线程池的方法
	 *
	 */
	public class ThreadPool {
		public static void main(String[] args){
			//使用Executors工具类中的方法创建线程池
			ExecutorService pool = Executors.newFixedThreadPool(5);
			ThreadPoolDemo demo = new ThreadPoolDemo();
			//为线程池中的线程分配任务,使用submit方法，传入的参数可以是Runnable的实现类，也可以是Callable的实现类
			for(int i=1;i<=5;i++){
				pool.submit(demo);
			}
			//关闭线程池
			//shutdown ： 以一种平和的方式关闭线程池，在关闭线程池之前，会等待线程池中的所有的任务都结束，不在接受新任务
			//shutdownNow ： 立即关闭线程池
			pool.shutdown();
		}
	}
	class ThreadPoolDemo implements Runnable{
		/**多线程的共享数据*/
		private int i = 0;
		@Override
		public void run() {
			while(i<=50){
				System.out.println(Thread.currentThread().getName()+"---"+ i++);
			}
		}
	}
```
实现callable然后加入线程池
``` Java 
	public class ThreadPool2 {

		public static void main(String args[]){
			ExecutorService executorService = Executors.newFixedThreadPool(5);

			for(int i=0;i<5;i++){
				//线程池执行submit之后直接返回一个Future
				Future<Integer> future = executorService.submit(new Callable<Integer>() {
				//匿名内部类直接重写runnable方法
					@Override
					public Integer call() throws Exception {
						int result = 0;
						for(int i=0;i<=10;i++){
							result += i;
						}
						return result;
					}
				});
				try {
					System.out.println(Thread.currentThread().getName()+"--"+future.get());
				} catch (InterruptedException | ExecutionException e) {
					e.printStackTrace();
				}
			}
			executorService.shutdown();
		}
	}
```