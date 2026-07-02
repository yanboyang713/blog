---
title: "creating microservices with rust"
draft: false
---

## Why [Rust]({{< relref "20230220230005-rust.md" >}}) is a great tool for creating [microservices]({{< relref "20230220225947-micro_service.md" >}}) {#why-rust--20230220230005-rust-dot-md--is-a-great-tool-for-creating-microservices--20230220225947-micro-service-dot-md}

Rust is an up-to-date, powerful, and reliable language. However, choosing it to implement [microservices]({{< relref "20230220225947-micro_service.md" >}}) is not an obvious decision, because Rust is a system programming language that is often assigned to low-level software such as drivers or OS kernels. This is because you tend to have to write a lot of glue code or get into detailed algorithms with low-level concepts, such as pointers in system programming languages. This is not the case with Rust. As a Rust programmer, you've surely already seen how it can be used to create high-level abstractions with flexible language capabilities.


### Explicit versus implicit {#explicit-versus-implicit}

Up until recently, there hasn't been a well-established approach to using [Rust]({{< relref "20230220230005-rust.md" >}}) for writing asynchronous network applications. Previously, developers tended to use two styles: either explicit control structures to handle asynchronous operations or implicit context switching. The explicit nature of Rust meant that the first approach outgrew the second. Implicit context switching is used in concurrent programming languages such as Go, but this model does not suit Rust for a variety of reasons. First of all, it has design limitations and it's hard or even impossible to share implicit contexts between threads. This is because the standard Rust library uses thread-local data for some functions and the program can't change the thread environment safely. Another reason is that an approach with context switching has overheads and therefore doesn't follow the zero-cost abstractions philosophy because you would have a background runtime. Some modern libraries such as [actix]({{< relref "20230807091357-actix.md" >}}) provide a high-level approach similar to automatic context switching, but actually use explicit control structures for handling asynchronous operations.

[Network programming in Rust]({{< relref "20230807091037-network_programming_in_rust.md" >}}) has evolved over time. When Rust was released, developers could only use the standard library. This method was particularly verbose and not suitable for writing high-performance servers. This was because the standard library didn't contain any good asynchronous abstractions. Also, event [hyper]({{< relref "20230807091537-hyper.md" >}}), a good crate for creating HTTP servers and clients, processed requests in separate threads and could therefore only have a certain number of simultaneous connections.

The [mio]({{< relref "20230807095737-mio_rust.md" >}}) crate was introduced to provide a clear asynchronous approach to make high-performance servers. It contained functions to interact with asynchronous features of the operating system, such as epoll or kqueue, but it was still verbose, which made it hard to write modular applications.

The next abstraction layer over mio was a futures and [tokio]({{< relref "20230807095935-tokio.md" >}}) pair of crates. The futures crate contained abstractions for implementing delayed operations (like the defers concept in Twisted, if you're familiar with Python). It also contained types for assembling stream processors, which are reactive and work like a finite state machine.

Using the [futures]({{< relref "20230807100446-futures_rs.md" >}}) crate was a powerful way to implement high-performance and high-accuracy network software. However, it was a middleware crate, which made it hard to solve everyday tasks. It was a good base for rewriting crates such as [hyper]({{< relref "20230807091537-hyper.md" >}}), because these can use explicit asynchronous abstractions with full control.

The highest level of abstraction today are crates that use [futures]({{< relref "20230807100446-futures_rs.md" >}}), [tokio]({{< relref "20230807095935-tokio.md" >}}), and [hyper]({{< relref "20230807091537-hyper.md" >}}) crates, such as [rocket]({{< relref "20230807100837-rocket.md" >}}) or [actix-web]({{< relref "20230807091357-actix.md" >}}). Now, [rocket]({{< relref "20230807100837-rocket.md" >}}) includes high-level elements to construct a web server with the minimal amount of lines. [actix-web]({{< relref "20230807091357-actix.md" >}}) works as a set of actors when your software is broken down into small entities that interact with one another. There are many other useful crates, but we will start with [hyper]({{< relref "20230807091537-hyper.md" >}}) as a basis for developing web servers from scratch. Using this crate, we will be between low-level crates, such as futures, and high-level crates, such as rocket. This will allow us to understand both in detail.


### Minimal amount of runtime errors {#minimal-amount-of-runtime-errors}

There are many languages suitable for creating [microservices]({{< relref "20230220225947-micro_service.md" >}}), but not every language has a reliable design to keep you from making mistakes. Most interpreted dynamic languages let you write flexible code that decides on the fly which field of the object to get and which function to call. You can often even override the rules of function calling by adding meta-information to objects. This is vital in meta-programming or in cases where your data drives the behavior of the runtime.

The dynamic approach, however, has significant drawbacks for the software, which requires reliability rather than flexibility. This is because any inaccuracy in the code causes the application to crash. The first time you try to use [Rust]({{< relref "20230220230005-rust.md" >}}), you may feel that it lacks flexibility. This is not true, however; the difference is in the approach you use to achieve flexibility. With Rust, all your rules must be strict. If you create enough abstractions to cover all of the cases your application might face, you will get the flexibility you want.

Rust rookies who come from the JavaScript or the Python world might notice that they have to declare every case of serialization/deserialization of data, whereas with dynamic languages, you can simply unpack any input data to the free-form object and explore the content later. You actually have to check all cases of inconsistency during runtime and try and work out what consequences could be caused if you change one field and remove another. With Rust, the compiler checks everything, including the type, the existence, and the corresponding format. The most important thing here is the type, because you can't compile a program that uses incompatible types. With other languages, this sometimes leads to strange compilation errors such as a case where you have two types for the same crate but the types are incompatible because they were declared in different versions of the same crate. Only Rust protects you from shooting yourself in the foot in this way. In fact, different versions can have different rules of serialization/deserialization for a type, even if both declarations have the same data layout.


### Great performance {#great-performance}

[Rust]({{< relref "20230220230005-rust.md" >}}) is a system programming language. This means your code is compiled into native binary instructions for the processor and runs without unwanted overhead, unlike interpreters such as JavaScript or Python.

Rust also doesn't use a garbage collector and you can control all allocations of memory and the size of buffers to prevent overflow.

Another reason why Rust is so fast for microservices is that it has zero-cost abstractions, which means that most abstractions in the language weigh nothing. They turn into effective code during compilation without any runtime overhead. For network programming, this means that your code will be effective after compilation, that is, once you have added meaningful constructions in the source code.


### Minimal dependencies burden {#minimal-dependencies-burden}

Rust programs are compiled into a single binary without unwanted dependencies. It needs [libc]({{< relref "20230807105105-libc.md" >}}) or another dynamic library if you want to use OpenSSL or similar irreplaceable dependencies, but all Rust crates are compiled statically into your code.

You may think that the compiled binaries are quite large to be used as microservices. The word microservice, however, refers to the narrow logic scope, rather than the size. Even so, statically linked programs remain tiny for modern computers.

What benefits does this give you? You will avoid having to worry about dependencies. Each Rust microservice uses its own set of dependencies compiled into a single binary. You can even keep microservices with obsolete features and dependencies besides new microservices. In addition, Rust, in contrast with the Go programming language, has strict rules for dependencies. This means that the project resists breaking, even if someone forces an update of the repository with the dependency you need.

How does Rust compare to Java? Java has microframeworks for building microservices, but you have to carry all dependencies with them. You can put these in a fat Java ARchive (JAR), which is a kind of compiled code distribution in Java, but you still need Java Virtual Machine (JVM). Don't forget, too, that Java will load every dependency with a class loader. Also, Java bytecode is interpreted and it takes quite a while for the Just-In-Time (JIT) compilation to finish to accelerate the code. With Rust, bootstrapping dependencies don't take a long time because they are attached to the code during compilation and your code will work with the highest speed from the start since it was already compiled into native code.


### [eBPF]({{< relref "20230613171810-ebpf.md" >}}) {#ebpf--20230613171810-ebpf-dot-md}
