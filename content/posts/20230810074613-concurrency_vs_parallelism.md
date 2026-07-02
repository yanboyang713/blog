---
title: "Concurrency vs Parallelism"
draft: false
---

## Concurrency vs Parallelism {#concurrency-vs-parallelism}

The conversational meanings of the words “parallel” and “concurrent” are mostly synonymous, which is a source of significant confusion that extends even to the computer science literature.  Distinguishing between parallel and concurrent programming is very important because both pursue different goals at different conceptual levels.

-   Concurrency is about multiple tasks which start, run, and complete in overlapping time periods, in no specific order.
-   Parallelism is about multiple tasks or subtasks of the same task that literally run at the same time on a hardware with multiple computing resources like multi-core processor.

As you can see, concurrency and parallelism are similar but not identical.

-   [Concurrent programming]({{< relref "20230810074202-concurrent_programming.md" >}})
-   [Parallel programming]({{< relref "20230810074540-parallel_programming.md" >}})

The essence of the relationship between concurrency and parallelism is that concurrent computations can be parallelized without changing the correctness of the result, but concurrency itself does not imply parallelism. Furthermore, parallelism does not imply concurrency; it is often possible for an optimizer to take programs with no semantic concurrency and break them down into parallel components via such techniques as pipeline processing, wide vector SIMD operations, or divide and conquer.

As Rob Pike pointed out “Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once.” In a single-core CPU, you can have concurrency but not parallelism. But both go beyond the traditional sequential model in which things happen one at a time.

To get more idea about the distinction between concurrency and parallelism, consider the following points:

-   An application can be concurrent but not parallel, which means that it processes more than one task at the same time, but no two tasks are executing at the same time instant.
-   An application can be parallel but not concurrent, which means that it processes multiple sub-tasks of a single task at the same time.
-   An application can be neither parallel nor concurrent, which means that it processes one task at a time, sequentially, and the task is never broken into subtasks.
-   An application can be both parallel and concurrent, which means that it processes multiple tasks or subtasks of a single task concurrently at the same time (executing them in parallel)

Imagine you have a program that inserts values into a hash table. If you spread the insert operation between multiple cores, that’s parallelism. But coordinating access to the hash table is concurrency.


## Reference List {#reference-list}

1.  <https://freecontent.manning.com/concurrency-vs-parallelism/>
