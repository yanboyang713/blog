---
title: "rust configuration file"
tags: ["rust"]
draft: false
---

[rust]({{< relref "20230220230005-rust.md" >}})


## Reading the configuration from file {#reading-the-configuration-from-file}

Environment variables and command-line arguments are useful to add temporary change parameters for a single run. They are a more convenient way to configure servers to use configuration files. This approach doesn't conform to The Twelve-Factor App methodology, but it's useful in cases when you need to set long parameters.

There are many formats that can be used for configuration files. The popular ones include TOML, YAML, and JSON. We will use TOML, because it is widely used with the Rust programming language.


### Adding the TOML config {#adding-the-toml-config}

The TOML file format is implemented in the toml crate. It previously used the now-obsolete rustc-serialize crate, but the last few versions have used the serde crate for serialization and deserialization. We will use both the toml and the serde crates.


### Adding dependencies {#adding-dependencies}

We actually need not only the **serde** crate but also the **serde_derive** crate. Both crates help with the serialization struct in various serialization formats. Add all three crates to the dependencies list in Cargo.toml:

```toml
serde = "1.0"
serde_derive = "1.0"
toml = "0.4"
```

The full list of imports in the **main.rs** file contains the following:

```rust
use clap::{crate_authors, crate_description, crate_name, crate_version, Arg, App};
use dotenv::dotenv;
use hyper::{Body, Response, Server};
use hyper::rt::Future;
use hyper::service::service_fn_ok;
use log::{debug, info, trace, warn};
use serde_derive::Deserialize;
use std::env;
use std::io::{self, Read};
use std::fs::File;
use std::net::SocketAddr;
```

As you can see, we haven't imported the serde crate here. We won't use it directly in the code because it's necessary to use the serde_derive crate instead. We have imported all macros from the **serde_derive** crate, because the **serde** crate contains the Serialize and Deserialize traits and **serde_derive** helps us to derive these for our structs.

Microservices often need to serialize and deserialize data when interacting with the client. We will cover this topic in the next chapter.


### Declaring a struct for configuration {#declaring-a-struct-for-configuration}

We have now imported all the necessary dependencies and can declare our configuration file structure. Add the Config struct to your code:

```rust
#[derive(Deserialize)]
struct Config {
    address: SocketAddr,
}
```

This struct contains only one field with the address. You can add more, but remember that all fields have to implement the Deserialize trait. The serde crate already has implementations for standard library types. For our types, we have to derive the implementation of Deserialize with the macro of the **serde_derive** crate.

Everything is ready for us to read the configuration from the file.


### Reading the configuration file {#reading-the-configuration-file}

Our server will expect to find a configuration file in the current working folder with the name microservice.toml. To read a configuration and convert it to the Config struct, we need to find and read this file if it exists. Add the following code to the main function of the server:

```rust
let config = File::open("microservice.toml")
    .and_then(|mut file| {
        let mut buffer = String::new();
        file.read_to_string(&mut buffer)?;
        Ok(buffer)
    })
    .and_then(|buffer| {
        toml::from_str::<Config>(&buffer)
            .map_err(|err| io::Error::new(io::ErrorKind::Other, err))
    })
    .map_err(|err| {
        warn!("Can't read config file: {}", err);
    })
    .ok();
```

The preceding code is a chain of method calls that start with the File instance. We use the open method to open the file and provide the name microservice.toml. The call returns a Result, which we will process in the chain. At the end of the processing, we will convert it to an option using the ok method and ignore any errors that occur during the parsing of the config file. This is because our service also supports environment variables and command-line parameters and has defaults for unset parameters.

When the file is ready, we will try to convert it into a String. We created an empty string, called a buffer, and used the read_to_string method of the File instance to move all of the data into the buffer. This is a synchronous operation. It's suitable for reading a configuration but you shouldn't use it for reading files to send to the client, because it will lock the runtime of the server until the file is read.

After we have read the buffer variable, we will try to parse it as a TOML file into the Config struct. The toml crate has a from_str method in the root namespace of the crate. It expects a type parameter to deserialize and an input string. We use the Config struct for the output type and our buffer for the input. But there is a problem: the File uses io::Error for errors, but from_str uses toml::de:Error for the error type. We can convert the second type to io::Error to make it compatible with the chain of calls.

The penultimate part of the chain is the **map_err** method call. We use this to write any errors with the configuration file to logs. As you can see, we used the Warn level. Issues with the configuration file are not critical, but it is important to be aware of them because they can affect the configuration. This makes the microservices.toml file optional.


### Joining all values by a priority {#joining-all-values-by-a-priority}

Our server has four sources of address settings:

-   The configuration file
-   The environment variable
-   The command-line parameter
-   The default value

We have to join these in this order. It's simple to implement this using a set of options and using the or method to set a value if the option doesn't contain anything. Use the following code to get address values from all of the sources:

```rust
let addr = matches.value_of("address")
    .map(|s| s.to_owned())
    .or(env::var("ADDRESS").ok())
    .and_then(|addr| addr.parse().ok())
    .or(config.map(|config| config.address))
    .or_else(|| Some(([127, 0, 0, 1], 8080).into()))
    .unwrap();
```

At first, this code takes a value from the --address command-line parameter. If it doesn't contain any value, the code tries to get a value from the ADDRESS environment variable. After that, we try to parse a textual value to the socket address. If all these steps fail, we can try to get a value from the Config instance that we read from microservice.toml. We will use the default address value if the value wasn't set by a user. In the previous address-parsing code, we also parsed the default value from a string. In this code, we use a tuple to construct the SocketAddr instance. Since we are guaranteed to get a value, we unwrap the option to extract it.


### Creating and using the configuration file {#creating-and-using-the-configuration-file}

We can now create a configuration file and run the server. Create the microservice.toml file in the root folder of the project and add the following line to it:

```rust
address = "0.0.0.0:9876"
```

Compile and start the service and you will see it has bound to that address.
