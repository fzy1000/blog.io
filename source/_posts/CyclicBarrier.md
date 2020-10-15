---
title: CyclicBarrier
date: 2020-06-19 14:58:47
tags:
---
{% codeblock [title] [lang:language] [url] [link text] [additional options] %}
code snippet
{% endcodeblock %}


---

``` Java 

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.Callable;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
 
public class Test1 {
 
	private static ExecutorService service = Executors.newFixedThreadPool(4);
	
	public static void main(String[] args) {
		
		final Map<String, Integer> map = new HashMap<String, Integer>(4);
		/* 使用了CyclicBarrier来实现。这个类的功能就是指定特定的线程数，等到这些线程都执行完毕之后，才会执行它的await()方法后面的代码，如果在构造器里设定了一个线程类，那么会在业务线程执行完毕之后，先执行构造器里的线程，然后执行await方法后面的线程。*/
		CyclicBarrier cb = new CyclicBarrier(4, new Runnable(){
			public void run() {
				Integer count1 = map.get("1");
				Integer count2 = map.get("2");
				Integer count3 = map.get("3");
				Integer count4 = map.get("4");
				System.out.println("count= " + (count1 + count2 + count3 + count4));
			}
		});
		new Thread(new TestThread(map, "1", cb,1000)).start();
		new Thread(new TestThread(map, "2", cb,1000)).start();
		new Thread(new TestThread(map, "3", cb,1000)).start();
		new Thread(new TestThread(map, "4", cb,1000)).start();
		
		/*线程池实现方式*/
		int sum = 0;
		for(int i = 0; i < 4; i++) {
			Future<Integer> data = service.submit(new Callable<Integer>(){
	
				@Override
				public Integer call() throws Exception {
					int sum = 0;
					for(int i = 0; i < 1000; i++) {
						sum += i;
					}
					return sum;
				}
				
			});
			try {
				sum += data.get().intValue();
			} catch (InterruptedException e) {
				e.printStackTrace();
			} catch (ExecutionException e) {
				e.printStackTrace();
			}
		}
		System.out.println("count===" + sum);
	}
 
}
 
class TestThread implements Runnable {
 
	private Map<String, Integer> map;
	private String type;
	private CyclicBarrier cb;
	private int n;
	public TestThread(Map<String, Integer> map, String type, CyclicBarrier cb, int n) {
		this.map = map;
		this.type =type;
		this.cb = cb;
		this.n = n;
	}
	
	@Override
	public void run() {
		int sum = 0;
		for(int i = 0; i < n; i++) {
			sum += i;
		}
		map.put(type, sum);
		try {
			cb.await();
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (BrokenBarrierException e) {
			e.printStackTrace();
		}
	
	}
	
}

```