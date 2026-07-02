---
title: "rust Error Handling"
draft: false
---

## Error Handling {#error-handling}

Error handling in Rust is done using explicit control flow:

-   Functions that can have errors list this in their return type,
-   There are no exceptions.


## Panics {#panics}

Rust will trigger a panic if a fatal error happens at runtime:

```rust
fn main() {
    let v = vec![10, 20, 30];
    println!("v[100]: {}", v[100]);
}
```

```output
   Compiling playground v0.0.1 (/playground)
    Finished dev [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/playground`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 100', src/main.rs:3:28
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

-   Panics are for unrecoverable and unexpected errors.
    -   Panics are symptoms of bugs in the program.
-   Use non-panicking APIs (such as Vec::get) if crashing is not acceptable.


## Catching the Stack Unwinding {#catching-the-stack-unwinding}

By default, a panic will cause the stack to unwind. The unwinding can be caught:

```rust
use std::panic;

fn main() {
    let result = panic::catch_unwind(|| {
        println!("hello!");
    });
    assert!(result.is_ok());

    let result = panic::catch_unwind(|| {
        panic!("oh no!");
    });
    assert!(result.is_err());
}
```

```output
hello!
```

-   This can be useful in servers which should keep running even if a single request crashes.
-   This does not work if panic = 'abort' is set in your Cargo.toml.


## Structured Error Handling with Result {#structured-error-handling-with-result}

We have already seen the Result enum. This is used pervasively when errors are expected as part of normal operation:

```rust
use std::fs;
use std::io::Read;

fn main() {
    let file = fs::File::open("diary.txt");
    match file {
        Ok(mut file) => {
            let mut contents = String::new();
            file.read_to_string(&mut contents);
            println!("Dear diary: {contents}");
        },
        Err(err) => {
            println!("The diary could not be opened: {err}");
        }
    }
}
```

```output
The diary could not be opened: No such file or directory (os error 2)
```

**Notes**

-   As with Option, the successful value sits inside of Result, forcing the developer to explicitly extract it. This encourages error checking. In the case where an error should never happen, [unwrap()]({{< relref "20230806023152-rust_unwrap.md" >}}) or expect() can be called, and this is a signal of the developer intent too.
-   Result documentation is a recommended read. Not during the course, but it is worth mentioning. It contains a lot of convenience methods and functions that help functional-style programming.


## Propagating Errors with ? {#propagating-errors-with}

The try-operator ? is used to return errors to the caller. It lets you turn the common

```rust
match some_expression {
    Ok(value) => value,
    Err(err) => return Err(err),
}
```

into the much simpler

```rust
some_expression?
```

We can use this to simplify our error handling code:

```rust
use std::{fs, io};
use std::io::Read;

fn read_username(path: &str) -> Result<String, io::Error> {
    let username_file_result = fs::File::open(path);
    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(err) => return Err(err),
    };

    let mut username = String::new();
    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(err) => Err(err),
    }
}

fn main() {
    //fs::write("config.dat", "alice").unwrap();
    let username = read_username("config.dat");
    println!("username or error: {username:?}");
}
```

```output
username or error: Err(Os { code: 2, kind: NotFound, message: "No such file or directory" })
```

**Notes**

-   The username variable can be either Ok(string) or Err(error).
-   Use the fs::write call to test out the different scenarios: no file, empty file, file with username.
-   The return type of the function has to be compatible with the nested functions it calls. For instance, a function returning a Result&lt;T, Err&gt; can only apply the ? operator on a function returning a Result&lt;AnyT, Err&gt;. It cannot apply the ? operator on a function returning a Result&lt;T, OtherErr&gt; or an Option&lt;AnyT&gt;. Reciprocally, a function returning an Option&lt;T&gt; can only apply the ? operator on a function returning an Option&lt;AnyT&gt;.
    -   You can convert incompatible types into one another with the different Option and Result methods such as Option::ok_or, Result::ok, Result::err.


## Converting Error Types {#converting-error-types}

The effective expansion of ? is a little more complicated than previously indicated:

```rust
expression?
```

works the same as

```rust
match expression {
    Ok(value) => value,
    Err(err)  => return Err(From::from(err)),
}
```

The From::from call here means we attempt to convert the error type to the type returned by the function:

```rust
use std::error::Error;
use std::fmt::{self, Display, Formatter};
use std::fs::{self, File};
use std::io::{self, Read};

#[derive(Debug)]
enum ReadUsernameError {
    IoError(io::Error),
    EmptyUsername(String),
}

impl Error for ReadUsernameError {}

impl Display for ReadUsernameError {
    fn fmt(&self, f: &mut Formatter) -> fmt::Result {
        match self {
            Self::IoError(e) => write!(f, "IO error: {e}"),
            Self::EmptyUsername(filename) => write!(f, "Found no username in {filename}"),
        }
    }
}

impl From<io::Error> for ReadUsernameError {
    fn from(err: io::Error) -> ReadUsernameError {
        ReadUsernameError::IoError(err)
    }
}

fn read_username(path: &str) -> Result<String, ReadUsernameError> {
    let mut username = String::with_capacity(100);
    File::open(path)?.read_to_string(&mut username)?;
    if username.is_empty() {
        return Err(ReadUsernameError::EmptyUsername(String::from(path)));
    }
    Ok(username)
}

fn main() {
    //fs::write("config.dat", "").unwrap();
    let username = read_username("config.dat");
    println!("username or error: {username:?}");
}
```

```rust
username or error: Err(IoError(Os { code: 2, kind: NotFound, message: "No such file or directory" }))
```

**Notes**

-   The username variable can be either Ok(string) or Err(error).
-   Use the fs::write call to test out the different scenarios: no file, empty file, file with username.
-   It is good practice for all error types that don’t need to be no_std to implement std::error::Error, which requires Debug and Display. The Error crate for core is only available in nightly, so not fully no_std compatible yet.
-   It’s generally helpful for them to implement Clone and Eq too where possible, to make life easier for tests and consumers of your library. In this case we can’t easily do so, because io::Error doesn’t implement them.


## Deriving Error Enums {#deriving-error-enums}

The [thiserror](https://docs.rs/thiserror/latest/thiserror/) crate is a popular way to create an error enum like we did on the previous page:

```rust
use std::{fs, io};
use std::io::Read;
use thiserror::Error;

#[derive(Debug, Error)]
enum ReadUsernameError {
    #[error("Could not read: {0}")]
    IoError(#[from] io::Error),
    #[error("Found no username in {0}")]
    EmptyUsername(String),
}

fn read_username(path: &str) -> Result<String, ReadUsernameError> {
    let mut username = String::new();
    fs::File::open(path)?.read_to_string(&mut username)?;
    if username.is_empty() {
        return Err(ReadUsernameError::EmptyUsername(String::from(path)));
    }
    Ok(username)
}

fn main() {
    //fs::write("config.dat", "").unwrap();
    match read_username("config.dat") {
        Ok(username) => println!("Username: {username}"),
        Err(err)     => println!("Error: {err}"),
    }
}
```

```output
Error: Could not read: No such file or directory (os error 2)
```

**Notes**

-   thiserror’s derive macro automatically implements std::error::Error, and optionally Display (if the #[error(...)] attributes are provided) and From (if the #[from] attribute is added). It also works for structs.
-   It doesn’t affect your public API, which makes it good for libraries.


## Dynamic Error Types {#dynamic-error-types}

Sometimes we want to allow any type of error to be returned without writing our own enum covering all the different possibilities. std::error::Error makes this easy.

```rust
use std::fs;
use std::io::Read;
use thiserror::Error;
use std::error::Error;

#[derive(Clone, Debug, Eq, Error, PartialEq)]
#[error("Found no username in {0}")]
struct EmptyUsernameError(String);

fn read_username(path: &str) -> Result<String, Box<dyn Error>> {
    let mut username = String::new();
    fs::File::open(path)?.read_to_string(&mut username)?;
    if username.is_empty() {
        return Err(EmptyUsernameError(String::from(path)).into());
    }
    Ok(username)
}

fn main() {
    //fs::write("config.dat", "").unwrap();
    match read_username("config.dat") {
        Ok(username) => println!("Username: {username}"),
        Err(err)     => println!("Error: {err}"),
    }
}
```

```output
Error: No such file or directory (os error 2)
```

**Notes**

-   This saves on code, but gives up the ability to cleanly handle different error cases differently in the program. As such it’s generally not a good idea to use Box&lt;dyn Error&gt; in the public API of a library, but it can be a good option in a program where you just want to display the error message somewhere.


## Adding Context to Errors {#adding-context-to-errors}

The widely used [anyhow](https://docs.rs/anyhow/latest/anyhow/) crate can help you add contextual information to your errors and allows you to have fewer custom error types:

```rust
use std::{fs, io};
use std::io::Read;
use anyhow::{Context, Result, bail};

fn read_username(path: &str) -> Result<String> {
    let mut username = String::with_capacity(100);
    fs::File::open(path)
        .with_context(|| format!("Failed to open {path}"))?
        .read_to_string(&mut username)
        .context("Failed to read")?;
    if username.is_empty() {
        bail!("Found no username in {path}");
    }
    Ok(username)
}

fn main() {
    //fs::write("config.dat", "").unwrap();
    match read_username("config.dat") {
        Ok(username) => println!("Username: {username}"),
        Err(err)     => println!("Error: {err:?}"),
    }
}
```

```output
Error: Failed to open config.dat

Caused by:
    No such file or directory (os error 2)
```

**Notes**

-   anyhow::Result&lt;V&gt; is a type alias for Result&lt;V, anyhow::Error&gt;.
-   anyhow::Error is essentially a wrapper around Box&lt;dyn Error&gt;. As such it’s again generally not a good choice for the public API of a library, but is widely used in applications.
-   Actual error type inside of it can be extracted for examination if necessary.
-   Functionality provided by anyhow::Result&lt;T&gt; may be familiar to Go developers, as it provides similar usage patterns and ergonomics to (T, error) from Go.


## Reference List {#reference-list}

1.  <https://google.github.io/comprehensive-rust/error-handling.html>
