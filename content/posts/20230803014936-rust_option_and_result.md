---
title: "rust Option and Result"
draft: false
---

## Option and Result {#option-and-result}

The types represent optional data:

```rust
fn main() {
    let numbers = vec![10, 20, 30];
    let first: Option<&i8> = numbers.first();
    println!("first: {first:?}");

    let idx: Result<usize, usize> = numbers.binary_search(&10);
    println!("idx: {idx:?}");
}
```

```output
first: Some(10)
idx: Ok(0)
```

**Notes**

-   Option and Result are widely used not just in the standard library.
-   Option&lt;&amp;T&gt; has zero space overhead compared to &amp;T.
-   Result is the standard type to implement error handling as we will see on Day 3.
-   binary_search returns Result&lt;usize, usize&gt;.
    -   If found, Result::Ok holds the index where the element is found.
    -   Otherwise, Result::Err contains the index where such an element should be inserted.


## Reference List {#reference-list}

1.  <https://google.github.io/comprehensive-rust/std/option-result.html>
