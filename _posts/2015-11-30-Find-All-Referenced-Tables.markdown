---
layout: post
title:  "Monitor the application in JVisualVm"
date:   2015-11-30 15:53:00
categories: JVM
---

> *This article is to illustrate how to monitor the application with the help of JVisualVm*

I am developing a JavaFx test tool for back-end server APIs validity checking. In practice, I found the application is easy to hang after running for a while. Therefore, I use the JVisualVm to monitor the application and find out what's going wrong there.

Let's go more details.

#### 1. How to Install JVisualVM ?
JVisualVm is a standard part of Sun's JDK distribution since JDK 6 Update. To begin the tour following [this link](https://docs.oracle.com/javase/8/docs/technotes/guides/visualvm/intro.html)

#### 2. To monitor your application
It's convenient to use JVisualVM to monitor your application. Just click the application's name on the "Applications" list view on the left. You can see the CPU/Heap/Classes/Threads usage diagrams by clicking the "Monitor" tab for further exploration.
![Alt text](/resources/jvisualvm/monitor.PNG)

By monitoring the threads, I found a big problem that every time I open one of the UI, which is for history review, it creates a new thread and after closing the UI, the tread still keeps alive and never exit (as shown in the following figure). That would cause a MOM disaster if the ui is frequently in use.

![Alt text](/resources/jvisualvm/thread-trace.jpg)

So, I check carefully in the source of the UI, I found a thread is always created to read the data from a blocking queue periodically during initialization of the UI. But the problem is the thread is embedded with a while(true) loop which will never exit so that even its parent UI died, the thread will not be killed but continue the dead loop. Therefore, I use a poison pill to message the thread to stop.Concretely, that is by changing the "poison pill" (a default true volatile boolean value) to false while on close method of the UI is invoked thus the thread will terminate itself by leaving the dead loop (refer to the following sampe codes). Then I assign a name to the thread and trace it in JVisualVm. As expected, the thread exit after I close the UI (see the following figure), thus solving the problem!

```java
   private volatile boolean stop = true;

   @Override
   public void run() {
           while(!stop) {
                   //Do something there
           }

   }

```

![Alt text](/resources/jvisualvm/thread-trace-1.jpg)

***Reference***
> <http://www.javaworld.com/article/2072322/from-jconsole-to-visualvm.html>
