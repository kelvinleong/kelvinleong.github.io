---
layout: post
title:  "Java Comparator"
date:   2015-08-28 23:24:58
categories: Java
---

#### [Java] Sort a List
----

In the recent work, I have to retrieved a list of data with the following data structure.

```java
  class Entity{
      int id; //Primary key
      type fields;
  }
```
 However, the data retrieved are not in sequential order sorted by the ID. In order to maintain an ascending ordering by ID to show in the UI, a comparator is introduced to help.

 Let's say we have a list Entity and want to sort the list by its id. The following Java codes show the step.

 ```java
    List<Entity> entityList = getEntityList();

    Collections.sort(entityList, new comparator(){
          @Override
          public int compare(Entity o1, Entity o2){
              return (o1.getId() - o2.getId());
          }
    })
 ```

 For a descending order, just change the position o1 and o2.

> Parameters: ***o1*** - the first object to be compared.
***o2*** - the second object to be compared.

> Returns:
a negative integer, zero, or a positive integer as the first argument is less than, equal to, or greater than the second.

Reference

> <https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html>
