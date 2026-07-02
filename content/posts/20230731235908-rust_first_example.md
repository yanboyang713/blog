---
title: "rust first example"
draft: false
---

## Create Project {#create-project}

1.  Use cargo new exercise to create a new exercise/ directory for your code:

<!--listend-->

```console
(base) [yanboyang713@archlinux ~]$ cargo new exercise
     Created binary (application) `exercise` package
```

1.  Navigate into exercise/ and use cargo run to build and run your binary:

<!--listend-->

```console
(base) [yanboyang713@archlinux ~]$ cd exercise/
(base) [yanboyang713@archlinux exercise]$ cargo run
   Compiling exercise v0.1.0 (/home/yanboyang713/exercise)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33s
     Running `target/debug/exercise`
Hello, world!
```

1.  Replace the boiler-plate code in src/main.rs with your own code. For example, using the example on the previous page, make src/main.rs look like

<!--listend-->

```rust
fn main() {
    println!("Edit me!");
}
```

1.  Use **cargo run** to build and run your updated binary:

<!--listend-->

```console
(base) [yanboyang713@archlinux exercise]$ cargo run
   Compiling exercise v0.1.0 (/home/yanboyang713/exercise)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33s
     Running `target/debug/exercise`
Edit me!
```

1.  Use **cargo check** to quickly check your project for errors, use cargo build to compile it without running it. You will find the output in target/debug/ for a normal debug build. Use cargo build --release to produce an optimized release build in target/release/.
    **NOTE**: You can add dependencies for your project by editing Cargo.toml. When you run cargo commands, it will automatically download and compile missing dependencies for you.


## Hello World {#hello-world}

Let us jump into the simplest possible Rust program, a classic Hello World program:

```rust
fn main() {
    println!("Hello 🌍!");
}
```

What you see:

-   Functions are introduced with fn.
-   Blocks are delimited by curly braces like in C and C++.
-   The main function is the entry point of the program.
-   Rust has hygienic macros, println! is an example of this.
-   Rust strings are UTF-8 encoded and can contain any Unicode character.


## language characteristic {#language-characteristic}

-   Rust is very much like other languages in the C/C++/Java tradition. It is imperative and it doesn’t try to reinvent things unless absolutely necessary.
-   Rust is modern with full support for things like Unicode.
-   Rust uses macros for situations where you want to have a variable number of arguments (no [Function Overloading]({{< relref "20230801042559-rust_function_overloading.md" >}})).
-   Macros being ‘hygienic’ means they don’t accidentally capture identifiers from the scope they are used in. Rust macros are actually only partially [hygienic]({{< relref "20230801042838-rust_hygiene.md" >}}).
-   Rust is multi-paradigm. For example, it has powerful [Object-Oriented Programming Features]({{< relref "20230801043016-rust_object_oriented_programming_features.md" >}}), and, while it is not a functional language, it includes a range of [Functional Language Features]({{< relref "20230801043202-rust_functional_language_features.md" >}}).


## Reference List {#reference-list}

1.  <https://google.github.io/comprehensive-rust/cargo/running-locally.html>
2.  <https://google.github.io/comprehensive-rust/hello-world.html>
