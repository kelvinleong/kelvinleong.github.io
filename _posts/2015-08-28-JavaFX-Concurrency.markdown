---
layout: post
title:  "JavaFX Concurrency"
date:   2015-08-29 20:15:58
categories: Java
---

####[Java] Differences between Platform.runlater() & Task/Serivce in JavaFx
---

>Repost from: http://stackoverflow.com/questions/13784333/platform-runlater-and-task-javafx

Use Platform.runLater(...) for quick and simple operations and Task for complex and big operations .

Use case for Platform.runLater(...)
Use case for Task: Task Example in Ensemble App
Example: Why Can't we use Platform.runLater(...) for long calculations (Taken from below reference).

Problem: Background thread which just counts from 0 to 1 million and update progress bar in UI.

<li>Code using Platform.runLater(...):</li>

```java
final ProgressBar bar = new ProgressBar();
new Thread(new Runnable() {
    @Override public void run() {
    for (int i=1; i<=1000000; i++) {
        final int counter = i;
        Platform.runLater(new Runnable() {
            @Override public void run() {
                bar.setProgress(counter/1000000.0);
            }
        });
    }
}).start();
```

This is a hideous hunk of code, a crime against nature (and programming in general). First, you’ll lose brain cells just looking at this double nesting of Runnables. Second, it is going to swamp the event queue with little Runnables — a million of them in fact. Clearly, we needed some API to make it easier to write background workers which then communicate back with the UI.

<li>Code using Task :</li>

```java
Task task = new Task<Void>() {
    @Override public Void call() {
        static final int max = 1000000;
        for (int i = 1; i <= max; i++) {
            updateProgress(i, max);
        }
        return null;
    }
};

ProgressBar bar = new ProgressBar();
bar.progressProperty().bind(task.progressProperty());
new Thread(task).start();
```

>it suffers from none of the flaws exhibited in the previous code

***Reference*** : [Worker Threading in JavaFX 2.0](http://fxexperience.com/2011/07/worker-threading-in-javafx-2-0/)
