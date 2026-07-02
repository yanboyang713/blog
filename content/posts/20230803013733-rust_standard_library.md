---
title: "rust Standard Library"
draft: false
---

## Standard Library {#standard-library}

Rust comes with a standard library which helps establish a set of common types used by Rust library and programs. This way, two libraries can work together smoothly because they both use the same String type.

The common vocabulary types include:

-   [Option and Result]({{< relref "20230803014936-rust_option_and_result.md" >}}) types: used for optional values and [error handling]({{< relref "20230803014616-rust_error_handling.md" >}}).
-   [String]({{< relref "20230803015036-rust_string.md" >}}): the default string type used for owned data.
-   [Vec]({{< relref "20230802021432-rust_vec.md" >}}): a standard extensible vector.
-   [HashMap]({{< relref "20230803014900-rust_hashmap.md" >}}): a hash map type with a configurable hashing algorithm.
-   [Box]({{< relref "20230802021335-rust_box.md" >}}): an owned pointer for heap-allocated data.
-   [Rc]({{< relref "20230802021530-rust_rc.md" >}}): a shared reference-counted pointer for heap-allocated data.

**Notes**
In fact, Rust contains several layers of the Standard Library: core, alloc and std.
core includes the most basic types and functions that don’t depend on libc, allocator or even the presence of an operating system.
alloc includes types which require a global heap allocator, such as Vec, Box and Arc.
Embedded Rust applications often only use core, and sometimes alloc.


## Reference List {#reference-list}

1.  <https://google.github.io/comprehensive-rust/std.html>
2.  <https://github.com/Warrenren/inside-rust-std-library>
