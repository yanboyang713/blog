---
title: "rust Control Flow"
draft: false
---

## Control Flow {#control-flow}

As we have seen, if is an expression in Rust. It is used to conditionally evaluate one of two blocks, but the blocks can have a value which then becomes the value of the if expression. Other control flow expressions work similarly in Rust.


## Blocks {#blocks}

A block in Rust contains a sequence of expressions. Each block has a value and a type, which are those of the last expression of the block:

```rust
fn main() {
    let x = {
        let y = 10;
        println!("y: {y}");
        let z = {
            let w = {
                3 + 4
            };
            println!("w: {w}");
            y * w
        };
        println!("z: {z}");
        z - y
    };
    println!("x: {x}");
}
```

```output
y: 10
w: 7
z: 70
x: 60
```

If the last expression ends with ;, then the resulting value and type is ().

The same rule is used for functions: the value of the function body is the return value:

```rust
fn double(x: i32) -> i32 {
    x + x
}

fn main() {
    println!("doubled: {}", double(7));
}
```

```output
doubled: 14
```

**Notes**

-   The point of this slide is to show that blocks have a type and value in Rust.
-   You can show how the value of the block changes by changing the last line in the block. For instance, adding/removing a semicolon or using a return.


## if expressions {#if-expressions}

You use [if expressions](https://doc.rust-lang.org/reference/expressions/if-expr.html#if-expressions) exactly like if statements in other languages:

```rust
fn main() {
    let mut x = 10;
    if x % 2 == 0 {
        x = x / 2;
    } else {
        x = 3 * x + 1;
    }
}
```

In addition, you can use if as an expression. The last expression of each block becomes the value of the if expression:

```rust
fn main() {
    let mut x = 10;
    x = if x % 2 == 0 {
        x / 2
    } else {
        3 * x + 1
    };
}
```

**Notes**

-   Because if is an expression and must have a particular type, both of its branch blocks must have the same type. Consider showing what happens if you add ; after x / 2 in the second example.


## if let expressions {#if-let-expressions}

The [if let expression](https://doc.rust-lang.org/reference/expressions/if-expr.html#if-let-expressions) lets you execute different code depending on whether a value matches a pattern:

```rust
fn main() {
    let arg = std::env::args().next();
    if let Some(value) = arg {
        println!("Program name: {value}");
    } else {
        println!("Missing name?");
    }
}
```

```output
Program name: target/debug/playground
```

See [Pattern Matching]({{< relref "20230803003950-rust_pattern_matching.md" >}}) for more details on patterns in Rust.

**Notes**

-   Unlike match, if let does not have to cover all branches. This can make it more concise than match.
-   A common usage is handling Some values when working with Option.
-   Unlike match, if let does not support guard clauses for pattern matching.


## let-else {#let-else}

<https://doc.rust-lang.org/rust-by-example/flow_control/let_else.html>


## while loops {#while-loops}

The [while keyword](https://doc.rust-lang.org/reference/expressions/loop-expr.html#predicate-loops) works very similar to other languages:

```rust
fn main() {
    let mut x = 10;
    while x != 1 {
        x = if x % 2 == 0 {
            x / 2
        } else {
            3 * x + 1
        };
    }
    println!("Final x: {x}");
}
```

```output
Final x: 1
```


## while let loops {#while-let-loops}

Like with [if let](https://doc.rust-lang.org/reference/expressions/loop-expr.html#predicate-pattern-loops), there is a while let variant which repeatedly tests a value against a pattern:

```rust
fn main() {
    let v = vec![10, 20, 30];
    let mut iter = v.into_iter();

    while let Some(x) = iter.next() {
        println!("x: {x}");
    }
}
```

```output
x: 10
x: 20
x: 30
```

Here the iterator returned by v.into_iter() will return a Option&lt;i32&gt; on every call to next(). It returns Some(x) until it is done, after which it will return None. The while let lets us keep iterating through all items.

**Notes**

-   Point out that the while let loop will keep going as long as the value matches the pattern.
-   You could rewrite the while let loop as an infinite loop with an if statement that breaks when there is no value to unwrap for iter.next(). The while let provides syntactic sugar for the above scenario.


## for loops {#for-loops}

The [for loop](https://doc.rust-lang.org/std/keyword.for.html) is closely related to the [while let loops](#while-let-loops). It will automatically call into_iter() on the expression and then iterate over it:

```rust
fn main() {
    let v = vec![10, 20, 30];

    for x in v {
        println!("x: {x}");
    }

    for i in (0..10).step_by(2) {
        println!("i: {i}");
    }
}
```

```output
x: 10
x: 20
x: 30
i: 0
i: 2
i: 4
i: 6
i: 8
```

You can use break and continue here as usual.

**Notes**

-   Index iteration is not a special syntax in Rust for just that case.
-   (0..10) is a range that implements an Iterator trait.
-   step_by is a method that returns another Iterator that skips every other element.
-   Modify the elements in the vector and explain the compiler errors. Change vector v to be mutable and the for loop to for x in v.iter_mut().


## loop expressions {#loop-expressions}

Finally, there is a [loop keyword](https://doc.rust-lang.org/reference/expressions/loop-expr.html#infinite-loops) which creates an endless loop.

Here you must either break or return to stop the loop:

```rust
fn main() {
    let mut x = 10;
    loop {
        x = if x % 2 == 0 {
            x / 2
        } else {
            3 * x + 1
        };
        if x == 1 {
            break;
        }
    }
    println!("Final x: {x}");
}
```

```output
Final x: 1
```

**Notes**

-   Break the loop with a value (e.g. break 8) and print it out.
-   Note that loop is the only looping construct which returns a non-trivial value. This is because it’s guaranteed to be entered at least once (unlike while and for loops).


## match expressions {#match-expressions}

The [match keyword](https://doc.rust-lang.org/reference/expressions/match-expr.html) is used to match a value against one or more patterns. In that sense, it works like a series of if let expressions:

```rust
fn main() {
    match std::env::args().next().as_deref() {
        Some("cat") => println!("Will do cat things"),
        Some("ls")  => println!("Will ls some files"),
        Some("mv")  => println!("Let's move some files"),
        Some("rm")  => println!("Uh, dangerous!"),
        None        => println!("Hmm, no program name?"),
        _           => println!("Unknown program name!"),
    }
}
```

Like [if let expressions](#if-let-expressions), each match arm must have the same type. The type is the last expression of the block, if any. In the example above, the type is ().

See [Pattern Matching]({{< relref "20230803003950-rust_pattern_matching.md" >}}) for more details on patterns in Rust.
**Notes**

-   Save the match expression to a variable and print it out.
-   Remove .as_deref() and explain the error.
    -   std::env::args().next() returns an Option&lt;String&gt;, but we cannot match against String.
    -   as_deref() transforms an Option&lt;T&gt; to Option&lt;&amp;T::Target&gt;. In our case, this turns Option&lt;String&gt; into Option&lt;&amp;str&gt;.
    -   We can now use pattern matching to match against the &amp;str inside Option.


## break and continue {#break-and-continue}

-   If you want to exit a loop early, use [break](https://doc.rust-lang.org/reference/expressions/loop-expr.html#break-expressions),
-   If you want to immediately start the next iteration use [continue](https://doc.rust-lang.org/reference/expressions/loop-expr.html#continue-expressions).

Both continue and break can optionally take a label argument which is used to break out of nested loops:

```rust
fn main() {
    let v = vec![10, 20, 30];
    let mut iter = v.into_iter();
    'outer: while let Some(x) = iter.next() {
        println!("x: {x}");
        let mut i = 0;
        while i < x {
            println!("x: {x}, i: {i}");
            i += 1;
            if i == 3 {
                break 'outer;
            }
        }
    }
}
```

```output
x: 10
x: 10, i: 0
x: 10, i: 1
x: 10, i: 2
```

In this case we break the outer loop after 3 iterations of the inner loop.


## Reference List {#reference-list}

1.  <https://google.github.io/comprehensive-rust/control-flow.html>
