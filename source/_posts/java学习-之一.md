---
title: java学习 之一
date: 2016-07-18 14:22:24
tags:
 - Java
 - Java 基础
categories:
 - Java
---
<!--more-->

# 一个单词

> synchronized
 美  ['sɪŋkrənaɪzd]
 adj. 同步的；同步化的
 v. 使协调（synchronize的过去分词）；同时发生；校准

*看到这个单词就是泪,上次面试问到Java线程同步的问题,我说我一般使用RxBus...然后HR说:"我指的是最底层的实现",我其实知道这个单词(认识),但是不会读,实在想不起来,就问HR有没有什么提示,HR就说"有关键字,synchronize",我还是想不起来,不是因为我不知道啊,是因为我实在不认识这个单词,后来他提到同步锁我才想起来,赶紧挽回.最后还是被挂了,我的处女面*

<!--more-->

# 两个方法


## sleep()
> sleep()使当前线程进入停滞状态（阻塞当前线程），让出CUP的使用、目的是不让当前线程独自霸占该进程所获的CPU资源，以留一定时间给其他线程执行的机会;
　　 sleep()是Thread类的Static(静态)的方法；因此他不能改变对象的机锁，所以当在一个Synchronized块中调用Sleep()方法是，线程虽然休眠了，但是对象的机锁并木有被释放，其他线程无法访问这个对象（即使睡着也持有对象锁）。
　　在sleep()休眠时间期满后，该线程不一定会立即执行，这是因为其它线程可能正在运行而且没有被调度为放弃执行，除非此线程具有更高的优先级。

**所以说,sleep不会释放线程锁,它打盹也要持有对象锁.**

## wait()
> wait()方法是Object类里的方法；当一个线程执行到wait()方法时，它就进入到一个和该对象相关的等待池中，同时失去（释放）了对象的机锁（暂时失去机锁，wait(long timeout)超时时间到后还需要返还对象锁）；其他线程可以访问；
　　wait()使用notify或者notifyAlll或者指定睡眠时间来唤醒当前等待池中的线程。
　　wiat()必须放在synchronized block中，否则会在program runtime时扔出”java.lang.IllegalMonitorStateException“异常。

**所以说,wait以后会释放对象锁,其他线程可以访问该对象.**


``` java
/**
 * Created by Bigmercu on 16/7/18.
 * Email:bigmercu@gmail.com
 */

/**
 * 在主线程中调用普通方法,普通方法会被先执行.
 * */

public class Main implements Runnable {

	private static int num = 100;
	
	public void firstMethod(){
	    synchronized (this){
	        num += 10;
	        System.out.println(">>>" + num);
	    }
	}
	
	public void secondMethod() throws InterruptedException {
	    synchronized (this){
//            Thread.sleep(2000);
	        wait(2000);
	        num *= 10;
	    }
	}
	
	public void run() {
	    firstMethod();
	}
	
	public static void main(String args[]) throws InterruptedException {
	    Main main = new Main();
	    Thread thread = new Thread(main);
	    thread.start();
	    main.secondMethod();
	}
}

```


### 输出分析

情况一:使用 Thread.sleep(2000);
在暂停两秒以后输出 1010
说明先调用了secondMethod方法,执行该方法并没有释放线程锁

情况二:使用 wait(2000);
直接输出 110
还是先调用了secondMethod方法,但是该方法不再持有线程锁,执行了run方法,然后执行了firstMethod方法输出了num未被改变以前的值

结论:sleep在休眠时持有线程锁,但是wait不会.




## notify()
wait以后线程会进入线程池等待,当满足运行条件以后可以调用`notify`来唤醒线程.当有多线程等待可以调用`notifyAlll()`来唤醒所有线程.
> 因为wait()方法是通知当前线程等待并释放对象锁，notify()方法是通知等待此对象锁的线程重新获得对象锁，然而，如果没有获得对象锁，wait方法和notify方法都是没有意义的，即必须先获得对象锁，才能对对象锁进行操作，于是，才必须把notify和wait方法写到synchronized方法或是synchronized代码块中了。


``` java

public class Main implements Runnable {

	private static int num = 100;
	private boolean aBoolean = false;
	
	public void firstMethod() throws InterruptedException {
	    synchronized (this){
	        num += 10;
	        new Thread(new Runnable() {
	            public void run() {
	                try {
	                    secondMethod();
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }).start();
	        if(!aBoolean){
	            this.wait();
	        }
	        num -=5;
	        System.out.println(">>>" + num);
	    }
	}
	
	//在这个方法中使num增加
	public void secondMethod() throws InterruptedException {
	     synchronized (this){
	        while(num < 20000){
	            this.wait(1000);
	            if(num > 10000) {
	                //当达到我们要求时通知线程
	                this.notify();
	                aBoolean = true;
	            }
	            num += 3000;
	            System.out.println(">>>A" + num);
	        }
	    }
	
	}
	
	public void run() {
	    try {
	        firstMethod();
	    } catch (InterruptedException e) {
	        e.printStackTrace();
	    }
	}
	
	public static void main(String args[]) throws InterruptedException {
	    Main main = new Main();
	    Thread thread = new Thread(main);
	    thread.start();
	}
}
```
### 如果不notify

```		
 A3110
 A6110
 A9110
 A12110
 A15110
 A18110
 A21110
```

### notify

```
A3110
A6110
A9110
A12110
A15110
15105
A18105
A21105
```
