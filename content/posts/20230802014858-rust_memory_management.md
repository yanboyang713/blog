---
title: "rust Memory Management"
draft: false
---

## Memory Management {#memory-management}

Traditionally, languages have fallen into two broad categories:

-   Full control via manual memory management: C, C++, Pascal, …
-   Full safety via automatic memory management at runtime: Java, Python, Go, Haskell, …

Rust offers a new mix:

-   Full control and safety via compile time enforcement of correct memory management.

It does this with an explicit [Ownership]({{< relref "20230802015107-rust_ownership.md" >}}) concept.
First, let’s refresh how memory management works.


## The Stack vs The Heap {#the-stack-vs-the-heap}

Stack: Continuous area of memory for local variables.

-   Values have fixed sizes known at compile time.
-   Extremely fast: just move a stack pointer.
-   Easy to manage: follows function calls.
-   Great memory locality.

Heap: Storage of values outside of function calls.

-   Values have dynamic sizes determined at runtime.
-   Slightly slower than the stack: some book-keeping needed.
-   No guarantee of memory locality.


### Stack and Heap Example {#stack-and-heap-example}

Creating a String puts fixed-sized metadata on the stack and dynamically sized data, the actual string, on the heap:

```rust
fn main() {
    let s1 = String::from("Hello");
}
```

{{< figure src="https://res.cloudinary.com/dkvj6mo4c/image/upload/v1690955864/rust/stack_heap_fprowe.png" >}}

**Notes**

-   Mention that a String is backed by a Vec, so it has a capacity and length and can grow if mutable via reallocation on the heap.

We can inspect the memory layout with unsafe code. However, you should point out that this is rightfully unsafe!

```rust
fn main() {
    let mut s1 = String::from("Hello");
    s1.push(' ');
    s1.push_str("world");
    // DON'T DO THIS AT HOME! For educational purposes only.
    // String provides no guarantees about its layout, so this could lead to
    // undefined behavior.
    unsafe {
        let (ptr, capacity, len): (usize, usize, usize) = std::mem::transmute(s1);
        println!("ptr = {ptr:#x}, len = {len}, capacity = {capacity}");
    }
}
```

```output
ptr = 0x563eb406d9d0, len = 11, capacity = 20
```


## Manual Memory Management {#manual-memory-management}

You allocate and deallocate heap memory yourself.

If not done with care, this can lead to crashes, bugs, security vulnerabilities, and memory leaks.


### C Example {#c-example}

You must call free on every pointer you allocate with malloc:

```c
void foo(size_t n) {
    int* int_array = malloc(n * sizeof(int));
    //
    // ... lots of code
    //
    free(int_array);
}
```

Memory is leaked if the function returns early between malloc and free: the pointer is lost and we cannot deallocate the memory. Worse, freeing the pointer twice, or accessing a freed pointer can lead to exploitable security vulnerabilities.


## Scope-Based Memory Management {#scope-based-memory-management}

Constructors and destructors let you hook into the lifetime of an object.
By wrapping a pointer in an object, you can free memory when the object is destroyed. The compiler guarantees that this happens, even if an exception is raised.
This is often called resource acquisition is initialization (RAII) and gives you smart pointers.


### C++ Example {#c-plus-plus-example}

```c++
void say_hello(std::unique_ptr<Person> person) {
  std::cout << "Hello " << person->name << std::endl;
}
```

-   The std::unique_ptr object is allocated on the stack, and points to memory allocated on the heap.
-   At the end of say_hello, the std::unique_ptr destructor will run.
-   The destructor frees the Person object it points to.

Special move constructors are used when passing ownership to a function:

```c++
std::unique_ptr<Person> person = find_person("Carla");
say_hello(std::move(person));
```


## Automatic Memory Management {#automatic-memory-management}

An alternative to manual and scope-based memory management is automatic memory management:

-   The programmer never allocates or deallocates memory explicitly.
-   A garbage collector finds unused memory and deallocates it for the programmer.

Java Example
The person object is not deallocated after sayHello returns:

```java
void sayHello(Person person) {
  System.out.println("Hello " + person.getName());
}
```


## Memory Management in Rust {#memory-management-in-rust}

Memory management in Rust is a mix:

-   Safe and correct like Java, but without a garbage collector.
-   Depending on which abstraction (or combination of abstractions) you choose, can be a single unique pointer, reference counted, or atomically reference counted.
-   Scope-based like C++, but the compiler enforces full adherence.
-   A Rust user can choose the right abstraction for the situation, some even have no cost at runtime like C.

Rust achieves this by modeling ownership explicitly.
**Notes**

-   If asked how at this point, you can mention that in Rust this is usually handled by RAII wrapper types such as [Box]({{< relref "20230802021335-rust_box.md" >}}), [Vec]({{< relref "20230802021432-rust_vec.md" >}}), [Rc]({{< relref "20230802021530-rust_rc.md" >}}), or [Arc]({{< relref "20230802021618-rust_arc.md" >}}). These encapsulate ownership and memory allocation via various means, and prevent the potential errors in C.
-   You may be asked about destructors here, the [Drop]({{< relref "20230802021726-rust_drop.md" >}}) trait is the Rust equivalent.


## Comparison {#comparison}

Here is a rough comparison of the memory management techniques.


### Pros of Different Memory Management Techniques {#pros-of-different-memory-management-techniques}

-   Manual like C:
    -   No runtime overhead.
-   Automatic like Java:
    -   Fully automatic.
    -   Safe and correct.
-   Scope-based like C++:
    -   Partially automatic.
    -   No runtime overhead.
-   Compiler-enforced scope-based like Rust:
    -   Enforced by compiler.
    -   No runtime overhead.
    -   Safe and correct.


### Cons of Different Memory Management Techniques {#cons-of-different-memory-management-techniques}

-   Manual like C:
    -   Use-after-free.
    -   Double-frees.
    -   Memory leaks.
-   Automatic like Java:
    -   Garbage collection pauses.
    -   Destructor delays.
-   Scope-based like C++:
    -   Complex, opt-in by programmer (on C++).
    -   Circular references can lead to memory leaks
    -   Potential runtime overhead
-   Compiler-enforced and scope-based like Rust:
    -   Some upfront complexity.
    -   Can reject valid programs.


## Reference List {#reference-list}

1.  <https://google.github.io/comprehensive-rust/memory-management.html>
