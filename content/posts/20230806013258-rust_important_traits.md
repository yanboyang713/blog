---
title: "rust Important Traits"
draft: false
---

## Important Traits {#important-traits}

We will now look at some of the most common traits of the [rust Standard Library]({{< relref "20230803013733-rust_standard_library.md" >}}):

-   [Iterator](https://doc.rust-lang.org/std/iter/trait.Iterator.html) and [IntoIterator](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html) used in for loops,
-   [From](https://doc.rust-lang.org/std/convert/trait.From.html) and [Into](https://doc.rust-lang.org/std/convert/trait.Into.html) used to convert values,
-   [Read](https://doc.rust-lang.org/std/io/trait.Read.html) and [Write](https://doc.rust-lang.org/std/io/trait.Write.html) used for IO,
-   [Add](https://doc.rust-lang.org/std/ops/trait.Add.html), [Mul](https://doc.rust-lang.org/std/ops/trait.Mul.html), … used for operator overloading, and
-   [Drop](https://doc.rust-lang.org/std/ops/trait.Drop.html) used for defining destructors.
-   [Default](https://doc.rust-lang.org/std/default/trait.Default.html) used to construct a default instance of a type.


## Iterators {#iterators}

You can implement the [Iterator](https://doc.rust-lang.org/std/iter/trait.Iterator.html) trait on your own types:

```rust
struct Fibonacci {
    curr: u32,
    next: u32,
}

impl Iterator for Fibonacci {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        let new_next = self.curr + self.next;
        self.curr = self.next;
        self.next = new_next;
        Some(self.curr)
    }
}

fn main() {
    let fib = Fibonacci { curr: 0, next: 1 };
    for (i, n) in fib.enumerate().take(5) {
        println!("fib({i}): {n}");
    }
}
```

```output
fib(0): 1
fib(1): 1
fib(2): 2
fib(3): 3
fib(4): 5
```

**Notes**

-   The Iterator trait implements many common functional programming operations over collections (e.g. map, filter, reduce, etc). This is the trait where you can find all the documentation about them. In Rust these functions should produce the code as efficient as equivalent imperative implementations.
-   IntoIterator is the trait that makes for loops work. It is implemented by collection types such as Vec&lt;T&gt; and references to them such as &amp;Vec&lt;T&gt; and &amp;[T]. Ranges also implement it. This is why you can iterate over a vector with for i in some_vec { .. } but some_vec.next() doesn’t exist.


## FromIterator {#fromiterator}

[FromIterator](https://doc.rust-lang.org/std/iter/trait.FromIterator.html) lets you build a collection from an Iterator.

```rust
fn main() {
    let primes = vec![2, 3, 5, 7];
    let prime_squares = primes
        .into_iter()
        .map(|prime| prime * prime)
        .collect::<Vec<_>>();
}
```

**Notes**

-   Iterator implements fn collect&lt;B&gt;(self) -&gt; B where B: FromIterator&lt;Self::Item&gt;, Self: Sized
-   There are also implementations which let you do cool things like convert an Iterator&lt;Item = Result&lt;V, E&gt;&gt; into a Result&lt;Vec&lt;V&gt;, E&gt;.


## From and Into {#from-and-into}

Types implement [From](https://doc.rust-lang.org/std/convert/trait.From.html) and [Into](https://doc.rust-lang.org/std/convert/trait.Into.html) to facilitate type conversions:

```rust
fn main() {
    let s = String::from("hello");
    let addr = std::net::Ipv4Addr::from([127, 0, 0, 1]);
    let one = i16::from(true);
    let bigger = i32::from(123i16);
    println!("{s}, {addr}, {one}, {bigger}");
}
```

```output
hello, 127.0.0.1, 1, 123
```

[Into](https://doc.rust-lang.org/std/convert/trait.Into.html) is automatically implemented when From is implemented:

```rust
fn main() {
    let s: String = "hello".into();
    let addr: std::net::Ipv4Addr = [127, 0, 0, 1].into();
    let one: i16 = true.into();
    let bigger: i32 = 123i16.into();
    println!("{s}, {addr}, {one}, {bigger}");
}
```

```output
hello, 127.0.0.1, 1, 123
```

**Notes**

-   That’s why it is common to only implement From, as your type will get Into implementation too.
-   When declaring a function argument input type like “anything that can be converted into a String”, the rule is opposite, you should use Into. Your function will accept types that implement From and those that only implement Into.


## Read and Write {#read-and-write}

Using [Read](https://doc.rust-lang.org/std/io/trait.Read.html) and [BufRead](https://doc.rust-lang.org/std/io/trait.BufRead.html), you can abstract over u8 sources:

```rust
use std::io::{BufRead, BufReader, Read, Result};

fn count_lines<R: Read>(reader: R) -> usize {
    let buf_reader = BufReader::new(reader);
    buf_reader.lines().count()
}

fn main() -> Result<()> {
    let slice: &[u8] = b"foo\nbar\nbaz\n";
    println!("lines in slice: {}", count_lines(slice));

    let file = std::fs::File::open(std::env::current_exe()?)?;
    println!("lines in file: {}", count_lines(file));
    Ok(())
}
```

```output
lines in slice: 3
lines in file: 14080
```

Similarly, [Write](https://doc.rust-lang.org/std/io/trait.Write.html) lets you abstract over u8 sinks:

```rust
use std::io::{Result, Write};

fn log<W: Write>(writer: &mut W, msg: &str) -> Result<()> {
    writer.write_all(msg.as_bytes())?;
    writer.write_all("\n".as_bytes())
}

fn main() -> Result<()> {
    let mut buffer = Vec::new();
    log(&mut buffer, "Hello")?;
    log(&mut buffer, "World")?;
    println!("Logged: {:?}", buffer);
    Ok(())
}
```

```output
Logged: [72, 101, 108, 108, 111, 10, 87, 111, 114, 108, 100, 10]
```


## The Drop Trait {#the-drop-trait}

Values which implement [Drop](https://doc.rust-lang.org/std/ops/trait.Drop.html) can specify code to run when they go out of scope:

```rust
struct Droppable {
    name: &'static str,
}

impl Drop for Droppable {
    fn drop(&mut self) {
        println!("Dropping {}", self.name);
    }
}

fn main() {
    let a = Droppable { name: "a" };
    {
        let b = Droppable { name: "b" };
        {
            let c = Droppable { name: "c" };
            let d = Droppable { name: "d" };
            println!("Exiting block B");
        }
        println!("Exiting block A");
    }
    drop(a);
    println!("Exiting main");
}
```

```output
Exiting block B
Dropping d
Dropping c
Exiting block A
Dropping b
Dropping a
Exiting main
```

**Notes**

-   Why doesn’t Drop::drop take self?
    Short-answer: If it did, std::mem::drop would be called at the end of the block, resulting in another call to Drop::drop, and a stack overflow!
-   Try replacing drop(a) with a.drop().


## The Default Trait {#the-default-trait}

[Default](https://doc.rust-lang.org/std/default/trait.Default.html) trait produces a default value for a type.

```rust
#[derive(Debug, Default)]
struct Derived {
    x: u32,
    y: String,
    z: Implemented,
}

#[derive(Debug)]
struct Implemented(String);

impl Default for Implemented {
    fn default() -> Self {
        Self("John Smith".into())
    }
}

fn main() {
    let default_struct = Derived::default();
    println!("{default_struct:#?}");

    let almost_default_struct = Derived {
        y: "Y is set!".into(),
        ..Derived::default()
    };
    println!("{almost_default_struct:#?}");

    let nothing: Option<Derived> = None;
    println!("{:#?}", nothing.unwrap_or_default());
}
```

```output
Derived {
    x: 0,
    y: "",
    z: Implemented(
        "John Smith",
    ),
}
Derived {
    x: 0,
    y: "Y is set!",
    z: Implemented(
        "John Smith",
    ),
}
Derived {
    x: 0,
    y: "",
    z: Implemented(
        "John Smith",
    ),
}
```

**Notes**

-   It can be implemented directly or it can be derived via #[derive(Default)].
-   A derived implementation will produce a value where all fields are set to their default values.
    -   This means all types in the struct must implement Default too.
-   Standard Rust types often implement Default with reasonable values (e.g. 0, "", etc).
-   The partial struct copy works nicely with default.
-   Rust standard library is aware that types can implement Default and provides convenience methods that use it.
-   the .. syntax is called [struct update syntax](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax)


## Add, Mul, … {#add-mul}

Operator overloading is implemented via traits in [std::ops:](https://doc.rust-lang.org/std/ops/index.html)

```rust
#[derive(Debug, Copy, Clone)]
struct Point { x: i32, y: i32 }

impl std::ops::Add for Point {
    type Output = Self;

    fn add(self, other: Self) -> Self {
        Self {x: self.x + other.x, y: self.y + other.y}
    }
}

fn main() {
    let p1 = Point { x: 10, y: 20 };
    let p2 = Point { x: 100, y: 200 };
    println!("{:?} + {:?} = {:?}", p1, p2, p1 + p2);
}
```

```output
Point { x: 10, y: 20 } + Point { x: 100, y: 200 } = Point { x: 110, y: 220 }
```

**Notes**

-   You could implement Add for &amp;Point. In which situations is that useful?
    Answer: Add:add consumes self. If type T for which you are overloading the operator is not Copy, you should consider overloading the operator for &amp;T as well. This avoids unnecessary cloning on the call site.
-   Why is Output an associated type? Could it be made a type parameter of the method?
    Short answer: Function type parameters are controlled by the caller, but associated types (like Output) are controlled by the implementor of a trait.
-   You could implement Add for two different types, e.g. impl Add&lt;(i32, i32)&gt; for Point would add a tuple to a Point.


## Closures {#closures}

Closures or lambda expressions have types which cannot be named. However, they implement special [Fn](https://doc.rust-lang.org/std/ops/trait.Fn.html), [FnMut](https://doc.rust-lang.org/std/ops/trait.FnMut.html), and [FnOnce](https://doc.rust-lang.org/std/ops/trait.FnOnce.html) traits:

```rust
fn apply_with_log(func: impl FnOnce(i32) -> i32, input: i32) -> i32 {
    println!("Calling function on {input}");
    func(input)
}

fn main() {
    let add_3 = |x| x + 3;
    println!("add_3: {}", apply_with_log(add_3, 10));
    println!("add_3: {}", apply_with_log(add_3, 20));

    let mut v = Vec::new();
    let mut accumulate = |x: i32| {
        v.push(x);
        v.iter().sum::<i32>()
    };
    println!("accumulate: {}", apply_with_log(&mut accumulate, 4));
    println!("accumulate: {}", apply_with_log(&mut accumulate, 5));

    let multiply_sum = |x| x * v.into_iter().sum::<i32>();
    println!("multiply_sum: {}", apply_with_log(multiply_sum, 3));
}
```

```output
Calling function on 10
add_3: 13
Calling function on 20
add_3: 23
Calling function on 4
accumulate: 4
Calling function on 5
accumulate: 9
Calling function on 3
multiply_sum: 27
```

**Notes**

-   An Fn (e.g. add_3) neither consumes nor mutates captured values, or perhaps captures nothing at all. It can be called multiple times concurrently.
-   An FnMut (e.g. accumulate) might mutate captured values. You can call it multiple times, but not concurrently.
-   If you have an FnOnce (e.g. multiply_sum), you may only call it once. It might consume captured values.
-   FnMut is a subtype of FnOnce. Fn is a subtype of FnMut and FnOnce. I.e. you can use an FnMut wherever an FnOnce is called for, and you can use an Fn wherever an FnMut or FnOnce is called for.
-   The compiler also infers Copy (e.g. for add_3) and Clone (e.g. multiply_sum), depending on what the closure captures.
-   By default, closures will capture by reference if they can. The move keyword makes them capture by value.
    ```rust
    fn make_greeter(prefix: String) -> impl Fn(&str) {
        return move |name| println!("{} {}", prefix, name)
    }

    fn main() {
        let hi = make_greeter("Hi".to_string());
        hi("there");
    }
    ```

    ```output
    Hi there
    ```


## Reference List {#reference-list}

1.  <https://google.github.io/comprehensive-rust/traits/important-traits.html>
