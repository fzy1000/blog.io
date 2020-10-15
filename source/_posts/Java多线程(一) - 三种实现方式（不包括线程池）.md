---
title: 'Java多线程(一) - 三种实现方式（不包括线程池）'
date: 2020-09-02 14:07:02
tags: 
- code 
- 多线程
- Java
category: Java小技巧
---

## 多线程的三种方法
1. 继承Thread类
2. 实现runnable
3. 实现callable

- 区别
	1. 第一个可以直接new之后start，后两个要new Thread（new MyClass）.start().
	2. 后面两个可以放到线程池里面，第一个不行。
	3. 后两个可以实现一个类然后用Thread类start好几个线程，这样可以共享资源，但是第一个就不行
	1. 后面两个都可以用函数式接口加箭头函数，第一个不行
	1. 最后一个有return值，其他的都不行
	1. 后两个可以避免Java单继承多实现出现的问题
	
#### 线程和进程
进程：每个进程都有独立的代码和数据空间（进程上下文），进程间的切换会有较大的开销，一个进程包含1--n个线程。（进程是资源分配的最小单位）

线程：同一类线程共享代码和数据空间，每个线程有独立的运行栈和程序计数器(PC)，线程切换开销小。（线程是cpu调度的最小单位）

每个线程和进程一样分为五个阶段：创建、就绪、运行、阻塞、终止。

#### 继承Thread类
继承Thread类，重写run方法，这种写法就会造成线程不安全，因为所有的线程都可以随时访问公共静态区
``` Java 
	package notes;
	public class ThreadDemo {
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			new MyThread("Thread1").start();
			new MyThread("Thread2").start();
			new MyThread("Thread3").start();
		}
	}
	class MyThread extends Thread{
		private String name;
		private int num = 5;
		public MyThread(String name){
			this.name = name;
		}
		@Override
		public void run() {
			while(num>0) {
				System.out.println(name + "runs, i = " + num);
				num--;
			}
		}
	}	
```

#### 实现runnable方法
> 实现Runnable接口，使得该类有了多线程类的特征。run（）方法是多线程程序的一个约定。所有的多线程代码都在run方法里面。Thread类实际上也是实现了Runnable接口的类。

继承Thread会造成很多不方便，runnable接口显然更优秀

1）：适合多个相同的程序代码的线程去处理同一个资源
2）：可以避免java中的单继承的限制
3）：增加程序的健壮性，代码可以被多个线程共享，代码和数据独立
4）：线程池只能放入实现Runable或callable类线程，不能直接放入继承Thread的类

 
``` Java 
public class ThreadDemo {
    public static void main(String[] args) throws InterruptedException {
        // 实现runnable接口的类可以创建一次之后创建很多线程，这样他们都会公用线程里的资源，这里的5不加static也是公用的
        MyThread mythread = new MyThread("mythread");
        new Thread(mythread).start();
        new Thread(mythread).start();
        new Thread(mythread).start();

        Thread.sleep(1);
        // Java8 Lambda expression
        new Thread(() -> {
            System.out.println("In Java8, Lambda expression" + Thread.currentThread());
        }).start();
    }
}

class MyThread implements Runnable {
    private String name;
    private AtomicInteger num = new AtomicInteger(30);

    public MyThread(String name) {
        this.name = name;
    }

    @Override
    public void run() {
        while (num.get() > 0) {
            num.addAndGet(-1);
            System.out.println(Thread.currentThread() + "runs, i = " + num);
        }
    }
```
#### 实现Callable接口
自定义一个实现Callable的类，必须指定泛型，因为他会有返回值

``` Java 
	public class ThreadDemo {
		public static void main(String[] args) {、
		//这里也可以用一个类对应很多个futuretask，但是继承thread就不行
			MyCallable myThread1 = new MyCallable("thread1");
			FutureTask<Integer> futuretask1 = new FutureTask<Integer>(myThread1);
			FutureTask<Integer> futuretask2 = new FutureTask<Integer>(myThread1);
			FutureTask<Integer> futuretask3 = new FutureTask<Integer>(myThread1);
			new Thread(futuretask1).start();
			new Thread(futuretask2).start();
			new Thread(futuretask3).start();
		}
	}

	class MyCallable implements Callable<Integer>{
		private int num = 5;
		private String name;
		public MyCallable(String name) {
			this.name = name;
		}
		@Override
		public Integer call() {
			while(num>0) {
				System.out.println(name + "runs, i = " + num);
				num--;
			}
			return num;
		}
	}
    ```
