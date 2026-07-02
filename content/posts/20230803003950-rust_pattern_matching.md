---
title: "rust Pattern Matching"
draft: false
---

## Pattern Matching {#pattern-matching}

The match keyword let you match a value against one or more patterns. The comparisons are done from top to bottom and the first match wins.

The patterns can be simple values, similarly to switch in C and C++:

```rust
fn main() {
    let input = 'x';

    match input {
        'q'                   => println!("Quitting"),
        'a' | 's' | 'w' | 'd' => println!("Moving around"),
        '0'..='9'             => println!("Number input"),
        _                     => println!("Something else"),
    }
}
```

```output
Something else
```

The _ pattern is a wildcard pattern which matches any value.

**Notes**

-   You might point out how some specific characters are being used when in a pattern
    -   | as an or
    -   .. can expand as much as it needs to be
    -   1..=5 represents an inclusive range
    -   _ is a wild card
-   It can be useful to show how binding works, by for instance replacing a wildcard character with a variable, or removing the quotes around q.
-   You can demonstrate matching on a reference.
-   This might be a good time to bring up the concept of irrefutable patterns, as the term can show up in error messages.


## Destructuring Enums {#destructuring-enums}

Patterns can also be used to bind variables to parts of your values. This is how you inspect the structure of your types. Let us start with a simple enum type:

```rust
enum Result {
    Ok(i32),
    Err(String),
}

fn divide_in_two(n: i32) -> Result {
    if n % 2 == 0 {
        Result::Ok(n / 2)
    } else {
        Result::Err(format!("cannot divide {n} into two equal parts"))
    }
}

fn main() {
    let n = 100;
    match divide_in_two(n) {
        Result::Ok(half) => println!("{n} divided in two is {half}"),
        Result::Err(msg) => println!("sorry, an error happened: {msg}"),
    }
}
```

```output
100 divided in two is 50
```

Here we have used the arms to destructure the Result value. In the first arm, half is bound to the value inside the Ok variant. In the second arm, msg is bound to the error message.

**Notes**

-   The if/else expression is returning an enum that is later unpacked with a match.
-   You can try adding a third variant to the enum definition and displaying the errors when running the code. Point out the places where your code is now inexhaustive and how the compiler tries to give you hints.


## Destructuring Structs {#destructuring-structs}

You can also destructure structs:

```rust
struct Foo {
    x: (u32, u32),
    y: u32,
}

#[rustfmt::skip]
fn main() {
    let foo = Foo { x: (1, 2), y: 3 };
    match foo {
        Foo { x: (1, b), y } => println!("x.0 = 1, b = {b}, y = {y}"),
        Foo { y: 2, x: i }   => println!("y = 2, x = {i:?}"),
        Foo { y, .. }        => println!("y = {y}, other fields were ignored"),
    }
}
```

```output
x.0 = 1, b = 2, y = 3
```

**Notes**

-   Change the literal values in foo to match with the other patterns.
-   Add a new field to Foo and make changes to the pattern as needed.
-   The distinction between a capture and a constant expression can be hard to spot. Try changing the 2 in the second arm to a variable, and see that it subtly doesn’t work. Change it to a const and see it working again.


## Destructuring Arrays {#destructuring-arrays}

You can destructure arrays, tuples, and slices by matching on their elements:

```rust
#[rustfmt::skip]
fn main() {
    let triple = [0, -2, 3];
    println!("Tell me about {triple:?}");
    match triple {
        [0, y, z] => println!("First is 0, y = {y}, and z = {z}"),
        [1, ..]   => println!("First is 1 and the rest were ignored"),
        _         => println!("All elements were ignored"),
    }
}
```

```output
Tell me about [0, -2, 3]
First is 0, y = -2, and z = 3
```

**Notes**

-   Destructuring of slices of unknown length also works with patterns of fixed length.
    ```rust
    fn main() {
        inspect(&[0, -2, 3]);
        inspect(&[0, -2, 3, 4]);
    }

    #[rustfmt::skip]
    fn inspect(slice: &[i32]) {
        println!("Tell me about {slice:?}");
        match slice {
            &[0, y, z] => println!("First is 0, y = {y}, and z = {z}"),
            &[1, ..]   => println!("First is 1 and the rest were ignored"),
            _          => println!("All elements were ignored"),
        }
    }
    ```

<!--listend-->

```output
Tell me about [0, -2, 3]
First is 0, y = -2, and z = 3
Tell me about [0, -2, 3, 4]
All elements were ignored
```

-   Create a new pattern using _ to represent an element.
-   Add more values to the array.
-   Point out that how .. will expand to account for different number of elements.
-   Show matching against the tail with patterns [.., b] and [a@..,b]


## Match Guards {#match-guards}

When matching, you can add a guard to a pattern. This is an arbitrary Boolean expression which will be executed if the pattern matches:

```rust
#[rustfmt::skip]
fn main() {
    let pair = (2, -2);
    println!("Tell me about {pair:?}");
    match pair {
        (x, y) if x == y     => println!("These are twins"),
        (x, y) if x + y == 0 => println!("Antimatter, kaboom!"),
        (x, _) if x % 2 == 1 => println!("The first one is odd"),
        _                    => println!("No correlation..."),
    }
}
```

```output
Tell me about (2, -2)
Antimatter, kaboom!
```

**Notes**

-   Match guards as a separate syntax feature are important and necessary when we wish to concisely express more complex ideas than patterns alone would allow.
-   They are not the same as separate if expression inside of the match arm. An if expression inside of the branch block (after =&gt;) happens after the match arm is selected. Failing the if condition inside of that block won’t result in other arms of the original match expression being considered.
-   You can use the variables defined in the pattern in your if expression.
-   The condition defined in the guard applies to every expression in a pattern with an |.


## Reference List {#reference-list}

1.  <https://google.github.io/comprehensive-rust/pattern-matching.html>
