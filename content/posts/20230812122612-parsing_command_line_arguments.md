---
title: "parsing command-line arguments"
tags: ["rust"]
draft: false
---

[rust]({{< relref "20230220230005-rust.md" >}})

-   clap-rs [clap] — A simple to use, full featured command-line argument parser
-   cliparser — Simple command line parser. build badge
-   docopt/docopt.rs [docopt] — A Rust implementation of DocOpt
-   google/argh [argh] — An opinionated Derive-based argument parser optimized for code size build badge
-   killercup/quicli [quicli] — quickly build cool CLI apps in Rust
-   ksk001100/seahorse [seahorse] — A minimal CLI framework written in Rust Build status
-   TeXitoi/structopt [structopt] — parse command line argument by defining a struct


## Parsing command-line arguments {#parsing-command-line-arguments}

Environment variables are useful for using with containers. If you use your application from a console or you want to avoid a conflict of names with other variables, you can use command-line parameters. This is a more conventional way for developers to set parameters to the program.

You can also get command-line arguments with the env module. This contains the args function, which returns an Args object. This object is not an array or vector, but it's iterable and you can use the for loop processes all command-line arguments:

```rust
for arg in env::args() {
    // Interpret the arg here
}
```

This variant may come in handy in simple cases. For parsing arguments with complex rules, however, you have to use a command-line argument parser. A good implementation of this is contained in the clap crate.


## Using the clap crate {#using-the-clap-crate}

To use the clap crate for parsing arguments, you have to build a parser and use it for arguments. To build a parser, you start by creating an instance of the App type. To use it, add all the necessary imports.


### Adding dependencies {#adding-dependencies}

Add a dependency to Cargo.toml:

```toml
clap = "2.32"
```

This crate provides useful macros for adding meta information about the program. These are as follows:

-   crate_name!: Returns the name of the crate
-   crate_version!: Returns the version of the crate
-   crate_authors!: Returns the list of authors
-   crate_description!: Provides the description of the crate

All information for these macros is taken from the Cargo.toml file.
Import the necessary types. We need two types, which are App and Arg, and the macros mentioned previously:

```rust
use clap::{crate_authors, crate_description, crate_name, crate_version, Arg, App};
```


### Building a parser {#building-a-parser}

The process of building a parser is quite simple. You will create an App instance and feed this type with the Arg instances. The App also has methods that can be used to set information about the application. Add the following code to the main function of our server:

```rust
let matches = App::new(crate_name!())
         .version(crate_version!())
         .author(crate_authors!())
         .about(crate_description!())
         .arg(Arg::with_name("address")
              .short("a")
              .long("address")
              .value_name("ADDRESS")
              .help("Sets an address")
              .takes_value(true))
         .arg(Arg::with_name("config")
              .short("c")
              .long("config")
              .value_name("FILE")
              .help("Sets a custom config file")
              .takes_value(true))
        .get_matches();
```

First, we create an App instance with a new method that expects the name of the crate. We provide this using the crate_name! macro. After that, we use the version, author, and about methods to set this data using the corresponding macros. We can chain these method calls, because every method consumes and returns the updated App object. When we set meta-information about the application, we have to declare the supported arguments with the arg method.

To add an argument, we have to create an Arg instance with the with_name method, provide the name, and set extra parameters using chaining-of-methods calls. We can set a short form of the argument with the short method and the long form with the long method. You can set the name of the value for the generated documentation using the value_name method. You can provide a description of an argument using the help method. The takes_value method is used to indicate that this argument requires a value. There is also a required method to indicate that an option is required, but we didn't use that here. All options are optional in our server.

We added the **--address** argument using these methods to set the address of the socket that we will use to bind the server. It also supports the short form a of the argument. We will read this value later.

The server will support the **--config** argument to set a configuration file. We have added this argument to the builder, but we will use it in the next section of this chapter.

After we create the builder, we call the get_matches method. This reads arguments with **std::env::args_os** and returns an **ArgMatches** instance, which we can use to get the values of the command-line parameters. We assign it to the matches local variable.

We should add the get_matches method before any logging call because it also prints help messages. We should avoid printing logs with the help description.


## Reading arguments {#reading-arguments}

To read arguments, ArgMatches contains a value_of method, where you add the name of a parameter. In this case, it is convenient to use constants to avoid typos. Extract the **--address** argument, and if this does not exist, then check the ADDRESS environment variable. This means that the command-line argument is a higher priority than the environment variable and you can override the parameters from the .env file with command-line parameters:

```rust
let addr = matches.value_of("address")
    .map(|s| s.to_owned())
    .or(env::var("ADDRESS").ok())
    .unwrap_or_else(|| "127.0.0.1:8080".into())
    .parse()
    .expect("can't parse ADDRESS variable");
```

In this code, we have converted all of the provided string references with the &amp;str type to solid String objects. This is useful if you want to use the object later in the code or if you need to move it elsewhere.


## Usage {#usage}

When you use the clap crate in your application, you can use command-line parameters to tweak it. The clap crate adds a --help argument, which the user can use to print information about all the arguments. This description was generated automatically by the crate, as can be seen in the following example:

```console

$ ./target/debug/random-service-with-args --help
random-service-with-env 0.1.0
Your Name
Rust Microservice

USAGE:
    random-service-with-env [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -a, --address <ADDRESS>    Sets an address
    -c, --config <FILE>        Sets a custom config file
```

Our application successfully printed the usage info: it provided us with all flags, options, and usage variants. If you need to add your own help description, you can use the help method of the App instance to set any string as a help message.

If you use the cargo run command, you can also set command-line parameters after the -- parameter. This means that it stops reading the run command and passes all remaining arguments to the running application:

```console
$ cargo run -- --help
```

You can now start the server and set an address using the --address parameter with value:

```console
$ cargo run -- --address 0.0.0.0:2345
```

The server has started and prints to the console:

```console
Finished dev [unoptimized + debuginfo] target(s) in 0.10s                                                                                             Running `target/debug/random-service-with-args --address '0.0.0.0:2345'`
 INFO 2018-07-26T04:23:52Z: random_service_with_env: Rand Microservice - v0.1.0
DEBUG 2018-07-26T04:23:52Z: random_service_with_env: Trying to bind server to address: 0.0.0.0:2345
 INFO 2018-07-26T04:23:52Z: random_service_with_env: Used address: 0.0.0.0:2345
DEBUG 2018-07-26T04:23:52Z: random_service_with_env: Run!
DEBUG 2018-07-26T04:23:52Z: tokio_reactor::background: starting background reactor
```


## How to add subcommands {#how-to-add-subcommands}

Some popular applications, such as cargo and docker, use subcommands to provide multiple commands inside a single binary. We can also support subcommands with the clap crate. A microservice might have two commands: one to run the server and one to generate a secret for the HTTP cookies. Take a look at the following code:

```rust

let matches = App::new("Server with keys")
    .setting(AppSettings::SubcommandRequiredElseHelp)
    .subcommand(SubCommand::with_name("run")
        .about("run the server")
        .arg(Arg::with_name("address")
            .short("a")
            .long("address")
            .takes_value(true)
            .help("address of the server"))
    .subcommand(SubCommand::with_name("key")
        .about("generates a secret key for cookies")))
    .get_matches();
```

Here, we have used two methods. The setting method tweaks the builder and you can set it with variants of the AppSettings enumeration. The **SubcommandRequiredElseHelp** method requires us to use subcommands or prints help message if no subcommands are provided. To add a subcommand, we use the subcommand method with the SubCommand instance that we created with the with_name method. A subcommand instance also has methods to set meta information about a subcommand, like we did with the App instance. Subcommands can also take arguments.

In the preceding example above, we added two subcommands—run, to run the server, and key, to generate secrets. You can use these when you start the application:

```bash
$ cargo run -- run --address 0.0.0.0:2345
```

We have two run arguments because the cargo has a command with the same name.
