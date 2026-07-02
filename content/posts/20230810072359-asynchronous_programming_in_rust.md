---
title: "Asynchronous Programming in Rust"
draft: false
---

## Why Async? {#why-async}

We all love how Rust empowers us to write fast, safe software. But how does asynchronous programming fit into this vision?

Asynchronous programming, or async for short, is a concurrent programming model supported by an increasing number of programming languages. It lets you run a large number of concurrent tasks on a small number of OS threads, while preserving much of the look and feel of ordinary synchronous programming, through the async/await syntax.


## Async vs other concurrency models {#async-vs-other-concurrency-models}

[Concurrent programming]({{< relref "20230810074202-concurrent_programming.md" >}}) is less mature and "standardized" than regular, sequential programming. As a result, we express concurrency differently depending on which concurrent programming model the language is supporting. A brief overview of the most popular concurrency models can help you understand how asynchronous programming fits within the broader field of concurrent programming:

-   OS threads don't require any changes to the programming model, which makes it very easy to express concurrency. However, synchronizing between threads can be difficult, and the performance overhead is large. Thread pools can mitigate some of these costs, but not enough to support massive IO-bound workloads.
-   Event-driven programming, in conjunction with callbacks, can be very performant, but tends to result in a verbose, "non-linear" control flow. Data flow and error propagation is often hard to follow.
-   Coroutines, like threads, don't require changes to the programming model, which makes them easy to use. Like async, they can also support a large number of tasks. However, they abstract away low-level details that are important for systems programming and custom runtime implementors.
-   The actor model divides all concurrent computation into units called actors, which communicate through fallible message passing, much like in distributed systems. The actor model can be efficiently implemented, but it leaves many practical issues unanswered, such as flow control and retry logic.

In summary, [asynchronous programming]({{< relref "20230810082215-asynchronous_programming.md" >}}) allows highly performant implementations that are suitable for low-level languages like [Rust]({{< relref "20230220230005-rust.md" >}}), while providing most of the ergonomic benefits of threads and coroutines.

[threads vs coroutines]({{< relref "20230810081213-threads_vs_coroutines.md" >}}) - What is the difference?


## Async in Rust vs other languages {#async-in-rust-vs-other-languages}

Although [asynchronous programming]({{< relref "20230810082215-asynchronous_programming.md" >}}) is supported in many languages, some details vary across implementations. Rust's implementation of async differs from most languages in a few ways:

-   Futures are inert in Rust and make progress only when polled. Dropping a future stops it from making further progress.
-   Async is zero-cost in Rust, which means that you only pay for what you use. Specifically, you can use async without heap allocations and dynamic dispatch, which is great for performance! This also lets you use async in constrained environments, such as embedded systems.
-   No built-in runtime is provided by Rust. Instead, runtimes are provided by community maintained crates.
-   Both single- and multithreaded runtimes are available in Rust, which have different strengths and weaknesses.


## Async vs threads in Rust {#async-vs-threads-in-rust}

The primary alternative to async in Rust is using OS threads, either directly through **std::thread** or indirectly through a thread pool. Migrating from threads to async or vice versa typically requires major refactoring work, both in terms of implementation and (if you are building a library) any exposed public interfaces. As such, picking the model that suits your needs early can save a lot of development time.

OS threads are suitable for a small number of tasks, since threads come with CPU and memory overhead. Spawning and switching between threads is quite expensive as even idle threads consume system resources. A thread pool library can help mitigate some of these costs, but not all. However, threads let you reuse existing synchronous code without significant code changes—no particular programming model is required. In some operating systems, you can also change the priority of a thread, which is useful for drivers and other latency sensitive applications.

Async provides significantly reduced CPU and memory overhead, especially for workloads with a large amount of IO-bound tasks, such as servers and databases. All else equal, you can have orders of magnitude more tasks than OS threads, because an async runtime uses a small amount of (expensive) threads to handle a large amount of (cheap) tasks. However, async Rust results in larger binary blobs due to the state machines generated from async functions and since each executable bundles an async runtime.

On a last note, asynchronous programming is not better than threads, but different. If you don't need async for performance reasons, threads can often be the simpler alternative.


### Example: Concurrent downloading {#example-concurrent-downloading}

In this example our goal is to download two web pages concurrently. In a typical threaded application we need to spawn threads to achieve concurrency:

```rust
fn get_two_sites() {
    // Spawn two threads to do work.
    let thread_one = thread::spawn(|| download("https://www.foo.com"));
    let thread_two = thread::spawn(|| download("https://www.bar.com"));

    // Wait for both threads to complete.
    thread_one.join().expect("thread one panicked");
    thread_two.join().expect("thread two panicked");
}
```

However, downloading a web page is a small task; creating a thread for such a small amount of work is quite wasteful. For a larger application, it can easily become a bottleneck. In async Rust, we can run these tasks concurrently without extra threads:

```rust
async fn get_two_sites_async() {
    // Create two different "futures" which, when run to completion,
    // will asynchronously download the webpages.
    let future_one = download_async("https://www.foo.com");
    let future_two = download_async("https://www.bar.com");

    // Run both futures to completion at the same time.
    join!(future_one, future_two);
}
```

Here, no extra threads are created. Additionally, all function calls are statically dispatched, and there are no heap allocations! However, we need to write the code to be asynchronous in the first place, which this book will help you achieve.


### Custom concurrency models in Rust {#custom-concurrency-models-in-rust}

On a last note, Rust doesn't force you to choose between threads and async. You can use both models within the same application, which can be useful when you have mixed threaded and async dependencies. In fact, you can even use a different concurrency model altogether, such as event-driven programming, as long as you find a library that implements it.


## The State of Asynchronous Rust {#the-state-of-asynchronous-rust}

Parts of async Rust are supported with the same stability guarantees as synchronous Rust. Other parts are still maturing and will change over time. With async Rust, you can expect:

-   Outstanding runtime performance for typical concurrent workloads.
-   More frequent interaction with advanced language features, such as lifetimes and pinning.
-   Some compatibility constraints, both between sync and async code, and between different async runtimes.
-   Higher maintenance burden, due to the ongoing evolution of async runtimes and language support.

In short, async Rust is more difficult to use and can result in a higher maintenance burden than synchronous Rust, but gives you best-in-class performance in return. All areas of async Rust are constantly improving, so the impact of these issues will wear off over time.


### Language and library support {#language-and-library-support}

While asynchronous programming is supported by Rust itself, most async applications depend on functionality provided by community crates. As such, you need to rely on a mixture of language features and library support:

-   The most fundamental traits, types and functions, such as the [Future Trait]({{< relref "20230810071548-future_trait.md" >}}) are provided by the standard library.
-   The async/await syntax is supported directly by the Rust compiler.
-   Many utility types, macros and functions are provided by the futures crate. They can be used in any async Rust application.
-   Execution of async code, IO and task spawning are provided by "async runtimes", such as [Tokio]({{< relref "20230807095935-tokio.md" >}}) and [async-std]({{< relref "20230810085725-async_std.md" >}}). Most async applications, and some async crates, depend on a specific runtime. See "[The Async Ecosystem]({{< relref "20230810085915-the_async_ecosystem.md" >}})" section for more details.

Some language features you may be used to from synchronous Rust are not yet available in async Rust. Notably, Rust does not let you declare async functions in traits. Instead, you need to use workarounds to achieve the same result, which can be more verbose.


## Compiling and debugging {#compiling-and-debugging}

For the most part, compiler- and runtime errors in async Rust work the same way as they have always done in Rust. There are a few noteworthy differences:


### Compilation errors {#compilation-errors}

Compilation errors in async Rust conform to the same high standards as synchronous Rust, but since async Rust often depends on more complex language features, such as lifetimes and pinning, you may encounter these types of errors more frequently.


### Runtime errors {#runtime-errors}

Whenever the compiler encounters an async function, it generates a state machine under the hood. Stack traces in async Rust typically contain details from these state machines, as well as function calls from the runtime. As such, interpreting stack traces can be a bit more involved than it would be in synchronous Rust.


### New failure modes {#new-failure-modes}

A few novel failure modes are possible in async Rust, for instance if you call a blocking function from an async context or if you implement the [Future Trait]({{< relref "20230810071548-future_trait.md" >}}) incorrectly. Such errors can silently pass both the compiler and sometimes even unit tests. Having a firm understanding of the underlying concepts, which this book aims to give you, can help you avoid these pitfalls.


## Compatibility considerations {#compatibility-considerations}

Asynchronous and synchronous code cannot always be combined freely. For instance, you can't directly call an async function from a sync function. Sync and async code also tend to promote different design patterns, which can make it difficult to compose code intended for the different environments.

Even async code cannot always be combined freely. Some crates depend on a specific async runtime to function. If so, it is usually specified in the crate's dependency list.

These compatibility issues can limit your options, so make sure to research which async runtime and what crates you may need early. Once you have settled in with a runtime, you won't have to worry much about compatibility.


## Performance characteristics {#performance-characteristics}

The performance of async Rust depends on the implementation of the async runtime you're using. Even though the runtimes that power async Rust applications are relatively new, they perform exceptionally well for most practical workloads.

That said, most of the async ecosystem assumes a multi-threaded runtime. This makes it difficult to enjoy the theoretical performance benefits of single-threaded async applications, namely cheaper synchronization. Another overlooked use-case is latency sensitive tasks, which are important for drivers, GUI applications and so on. Such tasks depend on runtime and/or OS support in order to be scheduled appropriately. You can expect better library support for these use cases in the future.


## Reference List {#reference-list}

1.  <https://rust-lang.github.io/async-book/01_getting_started/02_why_async.html>
