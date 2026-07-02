---
title: "threads vs coroutines"
draft: false
---

Short answer: With threads, the operating system switches running threads preemptively according to its scheduler, which is an algorithm in the operating system kernel. With coroutines, the programmer and programming language determine when to switch coroutines; in other words, tasks are cooperatively multitasked by pausing and resuming functions at set points, typically (but not necessarily) within a single thread.

Long answer: In contrast to threads, which are pre-emptively scheduled by the operating system, coroutine switches are cooperative, meaning the programmer (and possibly the programming language and its runtime) controls when a switch will happen.
A language that supports native threads can execute its threads (user threads) onto the operating system's threads (kernel threads). Every process has at least one kernel thread. Kernel threads are like processes, except that they share memory space in their owning process with all other threads in that process. A process "owns" all its assigned resources, like memory, file handles, sockets, device handles, etc., and these resources are all shared among its kernel threads.

The operating system scheduler is part of the kernel that runs each thread for a certain amount time (on a single processor machine). The scheduler allocates time (timeslicing) to each thread, and if the thread isn't finished within that time, the scheduler pre-empts it (interrupts it and switches to another thread). Multiple threads can run in parallel on a multi-processor machine, as each thread can be (but doesn't necessarily have to be) scheduled onto a separate processor.

On a single processor machine, threads are timesliced and preempted (switched between) quickly (on Linux the default timeslice is 100ms) which makes them concurrent. However, they can't be run in parallel (simultaneously), since a single-core processor can only run one thing at a time.

Coroutines and/or generators can be used to implement cooperative functions. Instead of being run on kernel threads and scheduled by the operating system, they run in a single thread until they yield or finish, yielding to other functions as determined by the programmer. Languages with generators, such as Python and ECMAScript 6, can be used to build coroutines. Async/await (seen in C#, Python, ECMAscript 7, Rust) is an abstraction built on top of generator functions that yield futures/promises.

In some contexts, coroutines may refer to stackful functions while generators may refer to stackless functions.

Fibers, lightweight threads, and green threads are other names for coroutines or coroutine-like things. They may sometimes look (typically on purpose) more like operating system threads in the programming language, but they do not run in parallel like real threads and work instead like coroutines. (There may be more specific technical particularities or differences among these concepts depending on the language or implementation.)

For example, Java had "green threads"; these were threads that were scheduled by the Java virtual machine (JVM) instead of natively on the underlying operating system's kernel threads. These did not run in parallel or take advantage of multiple processors/cores--since that would require a native thread! Since they were not scheduled by the OS, they were more like coroutines than kernel threads. Green threads are what Java used until native threads were introduced into Java 1.2.

Threads consume resources. In the JVM, each thread has its own stack, typically 1MB in size. 64k is the least amount of stack space allowed per thread in the JVM. The thread stack size can be configured on the command line for the JVM. Despite the name, threads are not free, due to their use resources like each thread needing its own stack, thread-local storage (if any), and the cost of thread scheduling/context-switching/CPU cache invalidation. This is part of the reason why coroutines have become popular for performance critical, highly-concurrent applications.

Mac OS will only allow a process to allocate about 2000 threads, and Linux allocates 8MB stack per thread and will only allow as many threads that will fit in physical RAM.

Hence, threads are the heaviest weight (in terms of memory usage and context-switching time), then coroutines, and finally generators are the lightest weight.


## Reference List {#reference-list}

1.  <https://stackoverflow.com/questions/1934715/difference-between-a-coroutine-and-a-thread>
