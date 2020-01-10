---
layout: post
title:  "Vertx Thread Blocked Deep Dive Analysis"
date:   2020-01-09 00:00:00
categories: Java
---

## The vertx stuck

As shown in the logs, the vertx event loop thread was blocked as it took too much time (the limit is 2000ms i.e., 2s).

```
02:49:45.094 WARN  [vertx-blocked-thread-checker] i.v.core.impl.BlockedThreadChecker: Thread Thread[vert.x-eventloop-thread-0,5,main] has been blocked for 2897 ms, time limit is 2000
02:49:46.094 WARN  [vertx-blocked-thread-checker] i.v.core.impl.BlockedThreadChecker: Thread Thread[vert.x-eventloop-thread-0,5,main] has been blocked for 3898 ms, time limit is 2000
02:49:47.094 WARN  [vertx-blocked-thread-checker] i.v.core.impl.BlockedThreadChecker: Thread Thread[vert.x-eventloop-thread-0,5,main] has been blocked for 4898 ms, time limit is 2000
```

### How vertx work?
Vertx is a reactor framework which hosting an event loop thread that handle coming events. One of the key principle for reactor pattern to keep in mind is that don't block the event loop! If you have events that would potentially take a long time to process, put it in a worker thread. This is something vertx highly recommended in the official documents. By using <emcode>vertx.executeBlocking</emcode> which vertx eventloop thread will instead delegate the event to a worker thread to follow up. Below diagram shows the work flow how event loop thread work with a woker thread. 

![Alt text](/resources/WSThreadBlock/vertx-work-flow.png)

So it seems some of events blocked the eventloop thread thus crashing the whole server. Having tracked the log stacks and code bases seriously, we identified the following part may cause eventloop block. The <emcode>UserSocketVerticle</emcode> is one of vertx verticles to start other components from the begining during server startup and we have 22 components! 

```java
@Component
public class UserSocketVerticle extends AbstractVerticle {

    private List<UserSocketVerticleComponent> components;

    public UserSocketVerticle(@Lazy List<UserSocketVerticleComponent> components) {
        this.components = components;
    }

    @Override
    public void start() throws Exception {
        super.start();
        components.forEach(c -> c.start(getVertx()));
    }
}
```

So to avoid blocking the event loop thread, we should delegate the startup process to a worker thread using vertx.executeBlokcing. Therefore, we refactor the following codes to:

```java
@Component
public class UserSocketVerticle extends AbstractVerticle {

    private List<UserSocketVerticleComponent> components;

    public UserSocketVerticle(@Lazy List<UserSocketVerticleComponent> components) {
        this.components = components;
    }

	@Override
    public void start() throws Exception {
        super.start();
        vertx.executeBlocking(f -> {
            components.stream().forEach(c -> {
                log.info("Starting verticle [verticle:{}]", c.getClass().getSimpleName());
                c.start(vertx);
            });
        }, r -> {});
    }
}
```

The fix seemed work for a while as expected that no more event loop blocking occurred. However, we found some other big troubles came later on. Well... the worker thread is blocked this time. OMG

```
0:21:37.642 WARN  [vertx-blocked-thread-checker] i.v.core.impl.BlockedThreadChecker: Thread Thread[vert.x-worker-thread-7,5,main] has been blocked for 6026202 ms, time limit is 60000
io.vertx.core.VertxException: Thread blocked
```

At the first glance, it seemed like the startup process starve the executeBlocking so intuitively we try delegating c.start(vertx) to another worker thread to help in this way:
```java
@Override
    public void start() throws Exception {
        super.start();
        vertx.executeBlocking(f -> {
            components.stream().forEach(c -> {
                log.info("Starting verticle [verticle:{}]", c.getClass().getSimpleName());
                vertx.executeBlocking(f -> c.start(vertx), r -> {});
            });
        }, r -> {});
    }
```

However, no luck. It did not resolve the starvation.

```
10:21:37.642 WARN  [vertx-blocked-thread-checker] i.v.core.impl.BlockedThreadChecker: Thread Thread[vert.x-worker-thread-7,5,main] has been blocked for 6026202 ms, time limit is 60000
io.vertx.core.VertxException: Thread blocked
at java.base@11/jdk.internal.misc.Unsafe.park(Native Method)
at java.base@11/java.util.concurrent.locks.LockSupport.park(LockSupport.java:194)
at java.base@11/java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:885)
at java.base@11/java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireShared(AbstractQueuedSynchronizer.java:1009)
at java.base@11/java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireShared(AbstractQueuedSynchronizer.java:1324)
at java.base@11/java.util.concurrent.locks.ReentrantReadWriteLock$ReadLock.lock(ReentrantReadWriteLock.java:738)
at org.springframework.data.mapping.context.AbstractMappingContext.getPersistentEntity(AbstractMappingContext.java:201)
at org.springframework.data.mapping.context.AbstractMappingContext.getPersistentEntity(AbstractMappingContext.java:172)
at org.springframework.data.mapping.context.AbstractMappingContext.getPersistentEntity(AbstractMappingContext.java:85)
at org.springframework.data.mapping.context.MappingContext.getRequiredPersistentEntity(MappingContext.java:70)
at org.springframework.data.mongodb.core.MongoTemplate.determineCollectionName(MongoTemplate.java:2540)
at org.springframework.data.mongodb.core.ExecutableFindOperationSupport$ExecutableFindSupport.getCollectionName(ExecutableFindOperationSupport.java:226)
at org.springframework.data.mongodb.core.ExecutableFindOperationSupport$ExecutableFindSupport.doFind(ExecutableFindOperationSupport.java:213)
at org.springframework.data.mongodb.core.ExecutableFindOperationSupport$ExecutableFindSupport.firstValue(ExecutableFindOperationSupport.java:158)
at org.springframework.data.mongodb.repository.query.AbstractMongoQuery.lambda$getExecution$4(AbstractMongoQuery.java:124)
at org.springframework.data.mongodb.repository.query.AbstractMongoQuery$$Lambda$810/0x000000080098f840.execute(Unknown Source)
at org.springframework.data.mongodb.repository.query.AbstractMongoQuery.execute(AbstractMongoQuery.java:97)
at org.springframework.data.repository.core.support.RepositoryFactorySupport$QueryExecutorMethodInterceptor.doInvoke(RepositoryFactorySupport.java:602)
at org.springframework.data.repository.core.support.RepositoryFactorySupport$QueryExecutorMethodInterceptor.invoke(RepositoryFactorySupport.java:590)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:185)
at org.springframework.data.projection.DefaultMethodInvokingMethodInterceptor.invoke(DefaultMethodInvokingMethodInterceptor.java:59)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:185)
at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:185)
at org.springframework.data.repository.core.support.SurroundingTransactionDetectorMethodInterceptor.invoke(SurroundingTransactionDetectorMethodInterceptor.java:61)
at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:185)
at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:212)
````

Having gone thru the logs, we are really confused by exceptions raised by spring mongo. Why the vertx blocking is related to mongodb? What's more tricky is that blocking did not ensue steadily. To idenity the cause, we need to reproduce the case steadily ... but we have no ideas how it went wrong thus trapped in the dead loop. Fortuantely, we came up with some hints. Firstly, the error happened mostly after restarting the server and was not found during normal session. Emmm so it seemed like something went wrong during startup process. Also, no matter how hard we tried in our local environment (from the web page), we cannot reproduce the error. Therefore we turned our eyes on mobile. Lucikly, we reproduced the case! We reproduced the case thru following steps. Step 1, login thru a test mobile device in dev environment and keep the session alive. Step 2, restart the service and track the logs. Bingo, the exceptions raise. We repeated step 1 and 2 to make sure that the trouble ensues steadily. Finally we identity that errors occur steadily following the mentioned two steps. So the problem seems to be more clear, server is blocked after restart while some mobile devices still keep its session alive.
But why web did not make the trouble? That push us to think about the communication flow of the two frontend

## The architecture of backend and how frontend communicate with it

![Alt text](/resources/WSThreadBlock/backend.png)

Our backend holds both http and websocket servers in a monolithic spring boot server as shown in the above diagram. Frontend will connect to websocket server then trigger some socket event also it would make restful calls thru http server. But web and mobile behave differently. From web, it has to wait for several restful api responses so as to render the page before connecting the web socket server. That said web will wait until http session is ready thereafter it connects to the websocket. So successful connection to http server always precedes attempts to connect websocket.

![Alt text](/resources/WSThreadBlock/web-flow.png)

However, mobile did not follow the mentioned pattern. Connecting to websocket and requesting restful apis work seperately. That said successful connection to http server does not always precedes attempts to connect websocket. It's possible that mobile devices try to connect to websocket before any restful apis call. 

![Alt text](/resources/WSThreadBlock/app-flow.png)

But let's think further why the precedence matters? Well, the mongo exceptions give us some hints. Further tracking the socket event that client is requesting, we saw that it requires some db query. And inside mongo's implementation the query will impose a lock but may endup getting no return and starving the thread. The reason maybe that spring did not finish its startup process for mongo thus the query failed and starve the thread. So to avoid the starvation, just make sure mongo is ready before websocket server is processing any socket event. Emmmm the cheapeast solution is to make sure websocket server is ready after http server is up as spring will make sure that everything is okay before starting the http web server.

Though we could also fix it by forcing mobile behaves as web does then the trouble will be gone. But seriously speaking, it's stupid. After all, they serve different natures and should be allowed to have their own behaviors. So finally, we add the <emcode>ApplicationReadyEvent</emcode> annotation on websocket server init which tells that websocket init will be triggered only when it receives the ApplicationReadyEvent signal.

```java
    @EventListener(ApplicationReadyEvent.class)
    public void startWS(){
        HttpServerOptions options = new HttpServerOptions();
        options.setMaxWebsocketFrameSize(maxFrameSize);
        options.setMaxWebsocketMessageSize(maxFrameSize * 4);
        vertx.createHttpServer(options)
                .websocketHandler(webSocketHandler())
                .listen(webSocketPort);
    }
```    

