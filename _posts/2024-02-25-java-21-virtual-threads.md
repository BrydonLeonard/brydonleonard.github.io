---
layout: post
title: "Java 21 - Virtual Threads"
date: 2024-02-26
categories: learning java
tags: java21
---

The latest LTS release of Java, Java 21, was released in September last year. I want to take some time to get familiar with its new features (as well as some of those in the earlier releases that I missed along the way). Today, let's take a look at virtual threads.
## Platform vs virtual threads

In the JVM, ***platform threads*** are thin wrappers around OS threads and there's a 1:1 mapping between the two. When you create a new thread in Java, the JVM makes a syscall to create a new OS thread, which is then associated to your JVM thread for its entire lifetime. In addition to that thread creation process being expensive, the number of OS threads (and therefore JVM platform threads) on a system is limited, which means that they need to be managed carefully.

***Virtual threads*** are a new type of thread in Java 21 that break that 1:1 mapping to OS threads and are managed by the JVM instead! The JVM manages a ForkJoinPool of underlying platform threads called ***carrier threads*** and schedules virtual threads to be mounted and executed on those carriers[^1]. When a virtual thread is blocked (say because it's waiting on some blocking I/O operation), it's unmounted from its carrier thread[^2], which frees up that carrier to start working on some *other* virtual thread.

![Semaphores vs fixed thread pools](/assets/images/2024-02-25-java-21-virtual-threads/virtualThread.png)

## A new mental model

Because platform threads are so expensive to create, they're treated like a precious resource, managed carefully, and pooled until there's work for them to do. Cheap virtual threads, on the other hand, can and *should* be created for individual computational tasks (like a single request or calculation). 

There is some nuance to what tasks work best as virtual threads, however. Virtual threads can't be unmounted just *anywhere* in their execution; they have to yield to the scheduler to indicate that they're ready to be unmounted and they'll only do so at specific *scheduling points*, such as locking or calling a blocking I/O operation. If you've got some long-running CPU-intensive task that's never going to yield, virtual threads likely won't make it perform _worse_, but you're not going to see any benefit using them[^3]. 

`synchronized` blocks and methods are particularly dangerous for virtual threads. While inside a `synchronized` block, virtual threads are ***pinned*** and blocked from yielding to the scheduler. If you need to coordinate your virtual threads, use semaphores and locks instead. In fact, since virtual threads map 1:1 to tasks, limiting their concurrency with a semaphore ends up being functionally equivalent to submitting work to a fixed threadpool:

![Semaphores vs fixed thread pools](/assets/images/2024-02-25-java-21-virtual-threads/virtualThreadSemaphore.png)

Lastly, because virtual threads tend to be so short lived, you really don't want to be doing expensive computation per-thread and caching the result in a `ThreadLocal`  since that'll happen for *every* virtual thread. [`ScopedValues`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ScopedValue.html) are intended as a replacement, but they're still in preview in Java 21. 

## Creating and configuring virtual threads

Virtual threads implement `java.lang.Thread` and are a drop-in replacement for platform threads in Java 21. You can now instantiate threads with one of: 

```
Thread.ofPlatform().start(myRunnable); // For platform threads
Thread.ofVirtual().start(myRunnable);  // For virtual threads
```

There's also a new executor service that creates one new virtual thread per task that's submitted to it:
```  
ExecutorService virtualThreadService = Executors.newVirtualThreadPerTaskExecutor();
```

The number of carrier threads in the JVM isn't configurable at runtime, but is controlled by two JVM options:

| Option                                   | Description                                                                                           |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `jdk.virtualThreadScheduler.parallelism` | The number of carrier threads created on startup.                                                     |
| `jdk.virtualThreadScheduler.maxPoolSize` | When all carrier threads are blocked, the scheduler will create more, up to a limit of `maxPoolSize`. |

## Conclusions and continuations

Before I close out, I thought it'd be interesting to take a peek behind virtual threads' curtain. If you're familiar with how Kotlin's (or any other language's, really) coroutines are implemented, virtual threads' will be very familiar. 

When a virtual thread is created, it's associated with a ***continuation***, a construct that keeps track of its current execution state. When the virtual thread reaches a scheduling point, yields, and is suspended, its continuation writes the carrier thread's stack to the heap so that it can be picked up again later. When a virtual threads is resumed, the continuation replaces the carrier thread's stack with the one written to the heap earlier and continues the virtual thread's execution.

## References

Some other useful reading for getting up to speed on virtual threads:

- [The Java 21 Virtual Threads docs](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html)
- [The Java 21 Threads reference docs have been updated to include virtual thread info](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Thread.html)
- [Inside Java - Managing Throughput with Virtual Threads](https://inside.java/2024/02/04/sip094/)
- [Inside Java - Networking I/O with Virtual Threads](https://inside.java/2021/05/10/networking-io-with-virtual-threads/)
- [The Wikipedia entry for continuations](https://en.wikipedia.org/wiki/Continuation)



[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbrydonleonard.github.io%2Flearning%2Fjava%2F2024%2F02%2F26%2Fjava-21-virtual-threads.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

## Footnotes

[^1]: If you've worked with Go's Goroutines or Kotlin's coroutines, this'll sound very familiar. 
[^2]: Unmounted threads are also called parked or suspended.
[^3]: The fact that you have less control over the carrier thread pool that over a platform thread pool one you create yourself could also make it harder to optimize those tasks.
