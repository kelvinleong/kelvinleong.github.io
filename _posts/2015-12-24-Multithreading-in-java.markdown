---
layout: post
title:  "Multithreading in Java"
date:   2015-12-24 14:27:00
categories: Multithreading
---

*This passage is to demonstrate the usage of thread and some concrete examples in practice.*

>Threads allow multiple activities to proceed concurrently. Concurrent programming is harder than single-threaded programming, because more things can go wrong, and failures can be hard to reproduce. But you can’t avoid concurrency. It is inherent in much of what we do, and a requirement if you are to obtain good performance from multicore processors, which are now commonplace. This chapter contains advice to help you write clear, correct, well-documented concurrent programs.

This is the preface copied from "Effective Java", of course, concurrent programming is somehow a nightmare for developer particularly when production issues occur. Multithreading is known as notoriously hard to debug and reproduce the error. But as stated, multithreading empower the performance by making good use of multicore processors. Therefore, it's inevitable for a programmer to take the challenges on multithreading programming.

### How to create a thread in Java?

Basically, there are two ways to create threads in Java programming. One is to "implement Runnable" and the other is to "extends Thread".

With implements Runnable:

~~~java
public class ThreadB implements Runnable {
    public void run() {
        //Code
    }
}
//Started with a "new Thread(threadB).start()" call
~~~
Or, with extends Thread:

```java
public class ThreadA extends Thread {
    public ThreadA() {
         super("ThreadA");
    }

    public void run() {
        //Code
    }
}
//Started with a "threadA.start()" call
```
This hierarchy can be shown in the following diagram.

![Alt text](/resources/multithreading/thread-runnable-erp.png)

As shown in the code example, both two methods actually embed the thread body in run() method but the semantic difference is that ThreadB implement the interface run() of Runnable while ThreadA override the method in Thread object. In practice, creating thread using ThreadA is straightforward for beginner to go but ThreadB is actually the preferred method to create a thread. The reason is that ***"implements Runnable"*** is more flexible as multi inheritance is not supported in Java. Say, you want to customize a timer control to update the time every 2 seconds by extending an implemented timer class. By this way, you cannot use "extend Thread" method due to the single inheritance restrict in Java.Instead, you can only make the timer to ***"implements Runnable"*** and override the run() method to specialize the thread's behavior.

### Start a thread

Thread should be started by invoking the native method start() to begin execution.The Java Virtual Machine then calls the run method of this thread to go.Directly invoking the run() method from main thread, the run() method goes onto the current call stack rather than at the beginning of a new call stack i.e., run() will serve as a function call instead by direct invoking.

**start()** method of Thread class is used to start a newly created thread. It performs following tasks:

* A new thread starts(with new callstack).

* The thread moves from New state to the Runnable state.

* When the thread gets a chance to execute, its target run() method will run.

To further understand the pitfall, read the following passage.
http://tutorials.jenkov.com/java-concurrency/creating-and-starting-threads.html#common-pitfall

### Data concurrency

Synchronization in java is the capability to control the access of multiple threads to any shared resource.

Java Synchronization is better option where we want to allow only one thread to access the shared resource.

Syntax to use synchronized method:

~~~java
public class MyObject {
  synchronized someMethod(object reference expression) {   
    //code block   
  }  
}
~~~

With the ***synchronized*** keyword, it can be guaranteed that only one thread can access method "someMethod()" which is to lock an object for any shared resources. But there is performance tradeoff by simply using synchronized keyword. Suppose you have 50 lines of code in your method, but you want to synchronize only 5 lines. Therefore, synchronized block is an alternative to solve the problem.

Syntax to use synchronized block

~~~java
synchronized (object reference expression) {   
  //code block   
}
~~~

Actually, the synchronized method is equivalent to the following code snippet:

~~~java
public class MyObject {
  void someMethod(object reference expression) {  
    synchronized(this){
      //code block   
    }
  }  
}
~~~

For further details read the following:
http://www.javatpoint.com/synchronization-in-java

### Inter-Thread Communication

Inter-thread communication is important in multithreading programming. Let's say there is a scenario that ThreadA is waiting for ThreadB completing its execution to continue its job. One of the solution is to create a status tracker which allows one or more threads to wait until a set of operations being performed in other threads completes.

~~~java
public class StatusTracker {
	public enum STATUS {
		PENDING,
		COMPLETED
	}		

	private ConcurrentMap<STATUS, CountDownLatch> signalLatchMap =
				  			new ConcurrentHashMap<STATUS, CountDownLatch>();
	private int status;

	public void await(STATUS signal) {
		try {

			CountDownLatch signalLatch = signalLatchMap.get(signal);
			if (signalLatch != null) {
				signalLatch.await();
				signalLatchMap.remove(signal);
			}
			else {
				signalLatch = new CountDownLatch(1);
				signalLatchMap.put(signal, signalLatch);
				signalLatch.await();
				signalLatchMap.remove(signal);
			}
		}
		catch (Exception e) { }
	}

	public void signal(STATUS signal) {
		CountDownLatch signalLatch = signalLatchMap.get(signal);
		if (signalLatch != null){
			signalLatch.countDown();
		}
	}
}
~~~

By invoking ***await(STATUS.COMPLETED)*** in ThreadA, it will release the lock and wait for the notification invoked by ***signal(STATUS.COMPLETED)*** in ThreadB.

For the further learning of CountDownLatch, read:
<https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/CountDownLatch.html>

***Reference***

> <https://docs.oracle.com/cd/B19306_01/server.102/b14237/statviews_1037.htm#i1576022>
> <http://www.javatpoint.com/multithreading-in-java>
