---
layout: post
title:  "Java False Sharing Detect"
date:   2016-06-10 10:08:00
categories: Java
---

One little-discussed performance implication of synchronization involves false sharing. That used to be a fairly obscure artifact of threaded programs, but as multi-core machines become the norm—and as other, more obvious, synchronization performance issues are addressed—false sharing is an increasingly important issue.

False sharing occurs because of the way in which CPUs handle their cache. Consider the data in this simple class:

```java
  public class DataHolder {
      public volatile long L1;
      public volatile long L2;
      public volatile long L3;
      public volatile long L4;
  }
```
Each of the long values here is stored in memory adjacent to each other—for example, L1 could be stored at memory location 0xF20. Then L2 would be stored in memory at 0xF28, L3 at 0xF2c, and so on. When it comes time for the program to operate on L2, it will load a relatively large amount of memory—for example, 128 bytes from location 0xF00 to 0xF80—into a cache line on one of the cores of (one of) the CPU(s). A second thread that wants to operate on L3 will load that same chunk of memory into a cache line on a different core.

Loading nearby values like that makes sense in most cases—if the application accesses one particular instance variable in an object, it is likely to access nearby instance variables. If they are already loaded into the core’s cache, then that memory access is very, very fast—a big performance win.

The downside to this scheme is that whenever the program updates a value in its local cache, that core must notify all the other cores that the memory in question has been changed. Those other cores must invalidate their cache lines and reload that data from memory.

Let's see what happens if the *DataHolder* class is heavily used by multiple threads:

~~~java

public class FalseSharing extends Thread{
	private static class DataHolder{
		private volatile long l1 = 0;
		private volatile long l2 = 0;
		private volatile long l3 = 0;
		private volatile long l4 = 0;
	}

	private static DataHolder dh = new DataHolder();
	private static long loops;

	public FalseSharing(Runnable r){
		super(r);
	}

	public static void TestVolatile() throws InterruptedException{
		loops = 100000000;
		FalseSharing[] tests = new FalseSharing[4];

		tests[0] = new FalseSharing(() -> {
		    for (long i = 0; i < loops; i++) {
			    dh.l1 += i;
		    }
		});

		tests[1] = new FalseSharing(() -> {
		    for (long i = 0; i < loops; i++) {
			    dh.l2 += i;
		    }
		});

		tests[2] = new FalseSharing(() -> {
		    for (long i = 0; i < loops; i++) {
			    dh.l1 += i;
		    }
		});

		tests[3] = new FalseSharing(() -> {
		    for (long i = 0; i < loops; i++) {
			    dh.l2 += i;
		    }
		});

		long then = System.currentTimeMillis();
    
		for (FalseSharing ct : tests) {
			ct.start();
		}
	  
		for (FalseSharing ct : tests) {
			ct.join();
		}

		long now = System.currentTimeMillis();
		System.out.println("Duration: " + (now - then) + " ms");
	}
}
~~~

There are four separate threads here, and they are not sharing any variables—each is accessing only a single member of the DataHolder class. From a synchronization standpoint, there is no contention, and we might reasonably expect that this code would execute in the same amount of time regardless of whether it runs one thread or four threads (given a four-core machine).

It doesn’t turn out that way—when one particular thread writes the volatile value in its loop, the cache line for every other thread will get invalidated, and the memory values must be reloaded

Let's see what happens if the *DataHolderLocal* (without volatile keyword) class is heavily used by multiple threads:

~~~java

public class FalseSharing extends Thread{
  	private static class DataHolderLocal{
  		private  long l1 = 0;
  		private  long l2 = 0;
  		private  long l3 = 0;
  		private  long l4 = 0;
  	}

  	private static DataHolder dh = new DataHolder();
  	private static DataHolderLocal dhLocal = new DataHolderLocal();
  	private static long loops;

  	public FalseSharing(Runnable r){
  		super(r);
  	}

  	public static void TestLocal() throws InterruptedException{
	    loops = 1000000;
	    FalseSharing[] tests = new FalseSharing[4];
		
	    tests[0] = new FalseSharing(() -> {
		    for (long i = 0; i < loops; i++) {
			dhLocal.l1 += i;
		    }
	    });

	    tests[1] = new FalseSharing(() -> {
	            for (long i = 0; i < loops; i++) {
	            	dhLocal.l2 += i;
	            }
	    });
	    
	    tests[2] = new FalseSharing(() -> {
		    for (long i = 0; i < loops; i++) {
			dhLocal.l1 += i;
            }
	    });
		
	    tests[3] = new FalseSharing(() -> {
	            for (long i = 0; i < loops; i++) {
	            	dhLocal.l2 += i;
	            }
	    });
	    
	    long then = System.currentTimeMillis();

	    for (FalseSharing ct : tests) {
		 ct.start();
	    }
	    
	    for (FalseSharing ct : tests) {
		ct.join();
	    }
	    
            long now = System.currentTimeMillis();
	    System.out.println("Duration: " + (now - then) + " ms");
	}
~~~

The following Table shows the result:


|loops         | Elapsed Time(volatile)         | Elapsed Time(wihout volatile)     |
|--------------|--------------------------------|-----------------------------------|
|100000000     |     6173ms                     |          142ms                    |
|1000000       |     607ms                      |           30ms                    |
|10000         |     61ms                       |           7ms                     |


The performance plunge dramatically due to involving volatile (or synchronzied) varaibles whereever any data value in the CPU cache is written, other caches that hold the same data range must be invalidated.

**Quick Summary:**

* False sharing can have significant performance impact on code that frequently modifies volatile variables or exits synchronized blocks.
* False sharing is difficult to detect. When a loop seems to be taking too long to occur, inspect the code to see if it matches the pattern where false sharing can occur.
* False sharing is best avoided by moving the data to local variables and storing them later. Alternately, padding can sometimes can be used to move the conflicting variables to different cache lines.
