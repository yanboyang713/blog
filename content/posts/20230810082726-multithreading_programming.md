---
title: "multithreading programming"
draft: false
---

## What Is Multithreading Programming? {#what-is-multithreading-programming}

Multithreading refers to the concurrent/parallel execution of more than one sequential set (thread) of instructions.

On a single processor, multithreading gives the illusion of running in parallel. In reality, the processor is switching by using a scheduling algorithm. Or, it’s switching based on a combination of external inputs (interrupts) and how the threads have been prioritized.

On multiple processor cores, threads are truly parallel. Individual microprocessors work together to achieve the result more efficiently. There are multiple parallel, concurrent tasks happening at once.

A basic example of multithreading is downloading two files from two different tabs in a web browser. Each tab uses a new thread to download the requested file. No tab waits for the other one to finish, they are downloading concurrently.

The following picture shows a simple explanation of concurrent execution of a multithreaded application:

{{< figure src="https://www.baeldung.com/wp-content/uploads/sites/4/2020/07/multithreading.png" >}}


## Reference List {#reference-list}

1.  <https://www.baeldung.com/cs/async-vs-multi-threading#what-is-multithreading-programming>
