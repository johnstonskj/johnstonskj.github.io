---
title: "Macros: Compile Time Magic (Rustioms)"
layout: postx
category: code
tags: [rust, macros, idioms]
---

True macros are an uncommon feature of systems languages although they've been a part of lispy languages for example
since the 1960's (see [MACRO Definitions for LISP](ftp://publications.ai.mit.edu/ai-publications/pdf/AIM-057.pdf), 
Timothy P. Hart, October 1963). The standard library has a bunch of macros that you'll see pretty often, `print!` and 
`println!`, `write!` and  `writeln!`, `panic!`, `assert!`, and `unimplemented!`. 

```rust
panic!("Something unexpected occurred");

assert!(value.is_some());

fn future_feature() {
    unimplemented!();
}
```

While we are all used to a bunch of `println!` calls for debugging, you can enhance messages with the location using
three useful macros. Also, consider the `dbg!` macro which prints the file location, the lexical form of the expression 
passed to it, and then the result of that expression. Very useful.

```rust
println!("in module {} (file {}), line {}", module_path!(), file!(), line!());

dbg!(start + (end - 1));
```

Results in the following.

```text
in module doc_examples::structs (file src/structs.rs), line 53

[src/structs.rs:58] start + (end - 1) = 30
```

There are also some macros you'll see less often, but which are really valuable. For example `env!` will return the 
value of an environment variable at compile time (and cargo sets a number of useful ones during a build), useful for 
caching version numbers.

```rust
const CRATE_NAME: &str = env!("CARGO_PKG_NAME");

const CRATE_VERSION: &str = env!("CARGO_PKG_VERSION");
```

The `include_str!` macro will actually read in a file's content at compile time; this is really useful for loading data 
resources, or for reading example data in tests. 

```rust
let xml = include_str!("../../../tests/example.xml");
```

## My First Macro

The real fun though is when you start to write your own, although I've used macros extensively in [Racket](https://racket-lang.org/)
I've not done much more than automate common patterns in code. Now, that doesn't mean that isn't valuable, when you 
can't easily factor work out into separate functions for whatever reason you can sometimes make use of closures to turn
code _inside-out_:

```rust
```

But, sometimes it's just ugly, or just as time-consuming and that's often where I've used Rust macros. 

## Documentation Links

* Official Documentation
  * [The Rust Programming Language](https://doc.rust-lang.org/book/ch19-06-macros.html)
  * [The Rust Reference](https://doc.rust-lang.org/reference/macros.html)
  * [Procedural Macros in Rust 2018](https://blog.rust-lang.org/2018/12/21/Procedural-Macros-in-Rust-2018.html)
  * [The Little Book of Rust Macros](https://danielkeep.github.io/tlborm/book/index.html) (not _really_ official)
* Articles
  * [A Beginner’s Guide to Rust Macros](https://medium.com/@phoomparin/a-beginners-guide-to-rust-macros-5c75594498f1)
  * [Rust Latam: procedural macros workshop](https://github.com/dtolnay/proc-macro-workshop)
  * [Creating Macros in Rust](https://hub.packtpub.com/creating-macros-in-rust-tutorial/)
  * [An Overview of Macros in Rust](https://words.steveklabnik.com/an-overview-of-macros-in-rust) **pre-2018**
