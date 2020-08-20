---
title: 'Java 回调函数'
date: 2020-08-19 16:37:02
tags: 
- code 
- 回调
- Java
category: Java小技巧
---

# 回调
回调函数，顾名思义就是指程序运行到代码A的时候，这段代码会去访问另一个scope的代码B，然后代码B的运行结果会影响到代码A的执行。这里我们有两种类型的回调函数，***同步回调*** 和 ***异步回调***。

# 同步回调
同步回调指的是在A代码调用了B代码后停止运行，一直等到代码B运行完成之后才会继续运行下去

# 异步回调
异步回调指的是在A代码调用了B代码之后会继续运行，并不会陷入阻塞状态，B代码运行结束后会影响到A代码

# 同步回调例子
我们举例说明，程序猿小冯在工作的时候，发现了一个程序猿小何写的bug，如果小何不修复这个bug，那么小冯就无法继续工作。这个时候小冯就去找小何帮忙修bug，等小何修完bug要告诉小冯的时候，这个时候就需要回调函数。小冯知道bug被修好了，那么就继续完成了自己的工作。

# 异步回调例子
程序猿小冯在工作的时候，发现了一个程序猿小何写的bug，这个时候小冯就去找小何帮忙修bug。在小何工作的同时，优秀员工小冯没有摸鱼，而是继续完成自己的工作。等小何的bug修好之后。再继续完成之前的工作。

# 代码
``` Java 
//回调函数，如果小何修完了bug就告诉小冯继续干活
interface callBack {
    void continueWork();
}
```
现在我们来实现小冯的代码
``` Java 
class Allen implements callBack {
    //因为需要和小何协调工作，所以小冯要认识小何这个对象
    private Scott scott;

    public Allen(Scott scott) {
        this.scott = scott;
    }

    //同步回调，意味着单线程工作，小冯把锅甩给了小何，如果小何不解决bug就不干活
    public void synwork() {
        System.out.println("小冯开始工作");
        shuaiguo();
        System.out.println("小冯工作完成");
    }

    //同步回调，意味着多线程完成，优秀员工小冯在小何修完bug之前就完成了自己这一部分的工作
    public void asywork() {
        System.out.println("小冯开始工作");
        new Thread(new Runnable() {
            @Override
            public void run() {
                shuaiguo();
            }
        }).start();
        System.out.println("小冯工作完成");
    }

    //这里就是甩锅的过程，小冯这里的特殊技能：叫小何干活
    public void shuaiguo() {
        scott.work(this);
    }

    //这里小冯实现了回调函数，等bug修好之后要继续干活，这个回调函数会通过类传给小何，这样小何就有了告诉小冯继续工作的方法了（小何本身并没有让小冯干活这项技能）
    @Override
    public void continueWork() {
        System.out.println("小冯继续工作");
    }
}

```

小冯的代码完成后，我们来写小何的代码

``` Java 
class Scott {
    //因为这个工作是被小冯通知要完成的，所以做完要告诉小冯，那么我们就把小冯这个对象传过来
    public void work(Allen allen) {
        System.out.println("Scott 开始工作");
        try {
            //scott完成工作需要的时间
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Scott 结束工作");
        //这里我们利用传过来的小冯这个对象完成回调
        allen.continueWork();
    }
}
```

这里我们来测试一下两种回调方法
``` Java 
public class test {
    public static void main(String[] args) {
        new Allen(new Scott()).synwork();
        new Allen(new Scott()).ayswork();
    }
}
```

## 异步回调结果
小冯开始工作
小冯工作完成
Scott 开始工作
Scott 结束工作
小冯继续工作

## 同步回调结果
小冯开始工作
Scott 开始工作
Scott 结束工作
小冯继续工作
小冯工作完成