---
title: "Simple Firewall with Rust and Aya"
draft: false
---

## Introduction {#introduction}

[eBPF]({{< relref "20230613171810-ebpf.md" >}}) and the Rust [Aya]({{< relref "20230810221450-aya.md" >}}) crate.
eBPF can run sandboxed program inside the kernel. Aya can create the byte code for both the kernel space programs, and the corresponding userland programs to load the eBPF byte code.
we will be looking at aya-rs, a rust interface to eBPF <https://aya-rs.dev/>


## Why Use eBPF and Aya? {#why-use-ebpf-and-aya}

Aya allows you to use eBPF, but also build both your userspace and kernel code with one command using Cargo. Testing and deployments are now much easier. There are other language bindings to use eBPF, several of which are relatively mature. Rust does have more stringent requirements during the compilation phase for code hygiene. This can only be a good thing when generating code that will be run with escalated privileges.


## Setting up the Prerequisites {#setting-up-the-prerequisites}

All the examples will be run on Ubuntu 24.04 LTS Linux.
Download Link: <https://mirror.umd.edu/ubuntu-iso/24.04/ubuntu-24.04-live-server-amd64.iso>


### setup dependencies {#setup-dependencies}

Install packages

```bash
sudo apt install clang llvm libelf-dev libpcap-dev build-essential libc6-dev-i386  \
graphviz  make gcc libssl-dev bc libelf-dev libcap-dev clang gcc-multilib  \
libncurses5-dev git pkg-config libmnl-dev bison flex linux-tools-$(uname -r)
```

Verify that you have \`bpftool\` installed on your system

```bash
sudo bpftool prog
```

If there are problems installing it from a package, you can install it from source:

```bash
$ git clone --recurse-submodules https://github.com/libbpf/bpftool.git
$ cd bpftool/src
$ make -j$(nproc)
$ sudo https://raw.githubusercontent.com/stevelatif/articles/main/blogs/bpftool prog
```

Install rust, following the instructions at <https://rustup.rs/>

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

Once you have rust and cargo installed and in your path, install the following rust related tools:

```bash
rustup update
cargo install cargo-generate
cargo install bpf-linker
```


## **XDP_pass** {#xdp-pass}

The eBPF program will run in a virtual machine that is built into the Linux kernel. This virtual machine has nine general purpose registers R1-R9 and one read only register R10 that functions as a frame pointer. Running anything that can be loaded dynamically in the kernel with elevated privileges can be potential security issue. The eBPF virtual machine contains a verifier see <https://docs.kernel.org/bpf/verifier.html> The verifier will check and reject if the program that contains:

loops
any type of pointer arithmetic
bounds or alignment violations
unreachable instructions
There are other restrictions, consult the documentation for more details

If you have worked with rust code with cargo before, you will have cycled through iterations of

```bash
cargo build
cargo run
```

Where the source tree of a simple application would like:

```console
$ tree
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

and after the application had been compiled:

```console
$ cargo build
   Compiling hello v0.1.0 (/tmp/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.07s
$ tree
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── target
    ├── CACHEDIR.TAG
    └── debug
        ├── build
        ├── deps
        │   ├── hello-20e1cfc616fb61ac
        │   └── hello-20e1cfc616fb61ac.d
        ├── examples
        ├── hello
        ├── hello.d
        └── incremental
            └── hello-3iozzekpyrysr
                ├── s-gvk87j1cfi-6yflog-9nq5zq3lt8rcg1slvtqhu2l92
                │   ├── 1cwggjlit3xor5e4.o
                │   ├── 3nmgwmthvx9ijli.o
                │   ├── 3qweoew2z2q7s3nh.o
                │   ├── 4j2x83e2uqqcjw4i.o
                │   ├── 4pluyvgu1vsjgq2o.o
                │   ├── 54dgri4zf1sqnocs.o
                │   ├── dep-graph.bin
                │   ├── query-cache.bin
                │   └── work-products.bin
                └── s-gvk87j1cfi-6yflog.lock

9 directories, 18 files
```

The compiled binary can be found in target/debug/hello and can be run directly from that location or by using cargo

```bash
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/hello`
Hello, world!
```

The aya framework creates two programs, an eBPF program that will be loaded into the kernel and user space program that will be load the eBPF program, and can also pass and receive data with the eBPF program using maps — more about these in later parts.

The code framework for the eBPF and user space will be set up using a template.
Move to next section, it is the first step.


## Generating the code {#generating-the-code}

Why use a template to generate the code? Aya-rs has been rapidly evolving, using older examples of code with the latest versions can lead to some errors that are hard to debug. Use the sample code here as a rough guide. Assuming that we have cargo and the generate extension have been installed, we can generate the code, at the prompt select xdp-pass as the project name

Using the template, generate the code in directory \`xdp-pass\`, select the xdp option.

```console
$ cargo generate https://github.com/aya-rs/aya-template
⚠️   Favorite `https://github.com/aya-rs/aya-template` not found in config, using it as a git repository: https://github.com/aya-rs/aya-template
🤷   Project Name: xdp-pass
🔧   Destination: /home/steve/articles/learning_ebpf_with_rust/xdp-tutorial/basic01-xdp-pass/xdp-pass ...
🔧   project-name: xdp-pass ...
🔧   Generating template ...
? 🤷   Which type of eBPF program? ›
  cgroup_skb
  cgroup_sockopt
  cgroup_sysctl
  classifier
  fentry
  fexit
  kprobe
  kretprobe
  lsm
  perf_event
  raw_tracepoint
  sk_msg
  sock_ops
  socket_filter
  tp_btf
  tracepoint
  uprobe
  uretprobe
❯ xdp
```

The generated files:

```console
yanboyang713@ebpf:~$ tree xdp-pass/
xdp-pass/
├── Cargo.toml
├── README.md
├── xdp-pass
│   ├── Cargo.toml
│   └── src
│       └── main.rs
├── xdp-pass-common
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── xdp-pass-ebpf
│   ├── Cargo.toml
│   ├── rust-toolchain.toml
│   └── src
│       └── main.rs
└── xtask
    ├── Cargo.toml
    └── src
        ├── build_ebpf.rs
        ├── build.rs
        ├── main.rs
        └── run.rs

9 directories, 14 files
```


### Explanation of the Directory Structure {#explanation-of-the-directory-structure}

-   xdp-pass/: The root of your project containing the main Cargo.toml and a README.
-   xdp-pass/: Contains the user space program.
    -   Cargo.toml: Manages dependencies for the user space program.
    -   src/main.rs: Entry point for the user space program.
-   xdp-pass-common/: Common code shared between user space and eBPF programs.
    -   Cargo.toml: Manages dependencies for the common library.
    -   src/lib.rs: Contains shared code.
-   xdp-pass-ebpf/: Contains the eBPF program.
    -   Cargo.toml: Manages dependencies for the eBPF program.
    -   rust-toolchain.toml: Specifies the Rust toolchain.
    -   src/main.rs: Entry point for the eBPF program.
-   xtask/: Contains custom tasks to extend Cargo's functionality.
    -   Cargo.toml: Manages dependencies for the xtask.
    -   src/build_ebpf.rs: Logic to build the eBPF program.
    -   src/main.rs: Entry point for the xtask.
    -   src/run.rs: Logic to run the eBPF program.


### Compile the code {#compile-the-code}

The eBPF program will be compiled and run using a [cargo workflow extension]({{< relref "2024-07-21-225902-cargo_workflow_extension.md" >}}):
Compilation is a two stage process:

```bash
cargo xtask build-ebpf
cargo build
```

Compile in this order else the \`cargo build\` will fail.


### Looking into the BPF-ELF object {#looking-into-the-bpf-elf-object}

eBPF bytecode is run in a virtual machine in the Linux kernel. There are 10 registers:

-   R0 stores function return values, and the exit value for an eBPF program
-   R1-R5 stores function arguments
-   R6-R9 are for general purpose usage
-   R10 stores adresses for the stack frame

Inspecting the sections of the eBPF file:


## Reference List {#reference-list}

1.  <https://medium.com/@stevelatif/simple-firewall-with-rust-and-aya-b56373c8bcc6>
