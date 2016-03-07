---
layout: post
title:  "Java Garbage Collection"
date:   2015-11-10 15:53:00
categories: Java
---

> *This article is to illustrate and explore the mechanism of Garbage Collection*

Garbage collection is a big topic for Java programmer and it always occurs in the interview (I was challenged several times) ! So in this passage, I will try to illustrate how Garbage Collection works with the Hotspot JVM.

Java's automatic garbage collection nicely frees programmers from worrying about memory management. Think about C++ programming, every time you new something, you have to cater for the delete appropriately. However, in java, you just new an object as you want without concerning to free them at all. But the convenience is not free and exactly JVM is responsible to collect the unreferenced objects(or so called "garbage"). That's why the interviewer like to challenge interviewee on this topic.

First, how does Garbage Collection works?

Java implements something called a generational garbage collector based upon the generational hypothesis assumption, which states that ***the majority of objects that are created are quickly discarded***, and objects that are not quickly collected are likely to be around for a while.

Let's have a look at the structure of Hotspot heap

![Alt text](/resources/Slide5.PNG)

As you can see from the diagram, the heap is divided into 3 parts, young , tenured and permanent generation respectively.

* The Young Generation is where all new objects are allocated and aged. When the young generation fills up, this causes a minor garbage collection. Minor collections can be optimized assuming a high object mortality rate. A young generation full of dead objects is collected very quickly. Some surviving objects are aged and eventually move to the old generation. More specifically,

  * Eden Space - Objects start out here. Most objects are created and destroyed in the Eden Space. Here, the GC does Minors GCs, which are optimized garbage collections. When a Minor GC is performed, any references to objects that are still needed are migrated to one of the survivors spaces (S0 or S1).

  * Survivor Space (S0 and S1) - Objects that survive Eden end up here. There are two of these, and only one is in use at any given time (unless we have a serious memory leak). One is designated as empty, and the other as live, alternating with every GC cycle.

* The Old Generation is used to store long surviving objects. Typically, a threshold is set for young generation object and when that age is met, the object gets moved to the old generation. Eventually the old generation needs to be collected. This event is called a major garbage collection.

* The Permanent generation contains metadata required by the JVM to describe the classes and methods used in the application. The permanent generation is populated by the JVM at runtime based on classes in use by the application. In addition, Java SE library classes and methods may be stored here.

***Reference***

> *  <http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html>
> *  <http://www.toptal.com/java/hunting-memory-leaks-in-java>
