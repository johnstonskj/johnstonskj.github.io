---
title: The Rust Toolchain (Rustioms)
layout: postx
category: code
tags: [rust, cargo, idioms]
---

Most languages these days focus a lot of time on ensuring a complete toolchain, not just a compiler. We see build tools, 
formatting, documentation, and package management front and center not add-ons. Rust is no different although the tools
are all extensible and can be customized for your process. The standard delivery mechanism is the crate, and the there
is a shared repository of these at [crates.io](https://crates.io).

Just things that make me feel good about the code I write, they generally make things more approachable. Some aren't 
even the language itself per-se, or even the toolchain, but the "Rust way". For example the great documentation tools 
that build the standard library, [docs.rustlang.org](https://doc.rust-lang.org/std/index.html), as well as all 3rd 
party crates on [docs.rs](https://docs.rs/). The integration between the documentation and testing tools so that all 
the examples embedded in documentation are treated as tests, à la Python 
[doctest](https://docs.python.org/3.9/library/doctest.html). I also really appreciate the fact that I don't have to 
argue about code style, or set coding conventions for every team or project I work on, there's one coding style, and 
there's a tool that enforces it, and any good build system rejects code that doesn't pass 
[rustfmt](https://github.com/rust-lang/rustfmt).


## Rustup

Technically optional, practically indispensable, [**rustup**](https://rustup.rs/) manages components of the core 
toolchain. You can add and remove toolchains (the set of everything needed to compile for a platform) using the 
`toolchain` command. You can also add individual components to the toolchain using the `component` command. The most
useful command though is `update`, this ensures the versions of all components in your installed toolchains are up to 
date.

Personally, I like to always have the **stable** and **nightly** toolchains to work with, and for ease of debugging 
the **rust-src** for the library and compiler. The **rustfmt** component should be a mandatory part of any build 
process, it formats all code to the same standards for all Rust programmers. Clippy on the other hand is the official, 
and at times obnoxious, lint tool; however it is also awesome, so run clippy regularly! Here's the setup script I use.

```zsh
#! /usr/bin/env zsh

curl  -sSf -o ./sh.rustup.rs https://sh.rustup.rs
sh ./sh.rustup.rs -y -v --no-modify-path

rustup toolchain add stable
rustup toolchain add nightly

rustup component add rust-src
rustup component add rustfmt
rustup component add clippy
rustup component add cargo

rustup update
```

## Cargo

A pluggable command-line tool to manage Rust projects, **[cargo](https://doc.rust-lang.org/cargo/)**, is a number of 
things:

* a build system with commands including `bench`, `build`, `check`, `clean`, `doc`, `fetch`, `fix`, `run`, & `test`;
* a package manager with commands including `install`, `new`, `search`, & `uninstall`;
* an interface to crates.io with commands including `package`, `publish`, & `yank`

It is also possible to install new commands for cargo, I tend to use:

* Essential
  * [**cargo-edit**](https://github.com/killercup/cargo-edit); This tool extends Cargo to allow you to add, remove, and 
    upgrade dependencies by modifying your `Cargo.toml` file from the command line.
  * [**cargo-make**](url:https://github.com/sagiegurari/cargo-make); Rust task runner and build tool.
  * [**cargo-update**](url:https://github.com/nabijaczleweli/cargo-update); Check for cargo installed executables' newer 
    versions and update as needed.
  * [**cargo-watch**](url:https://github.com/passcod/cargo-watch); Watch your repo for changes and build automatically.
* Release Cycle
  * [**cargo-audit**](https://github.com/RustSec/cargo-audit); Audit `Cargo.lock` files for crates with security 
    vulnerabilities reported to the [RustSec Advisory Database](url:https://github.com/RustSec/advisory-db/).
  * [**cargo-bloat**](url:https://github.com/RazrFalcon/cargo-bloat); Find out what takes most of the space in your 
    executable.
  * [**cargo-deadlinks**](url:https://github.com/deadlinks/cargo-deadlinks); Check your cargo doc documentation for 
    broken links.
  * [**cargo-grammarly**](url:https://github.com/vityafx/cargo-grammarly); Use the [grammarly](url:https://grammarly.com/) 
    service for checking English grammar in your crate's documentation.
  * [**cargo-outdated**](url:https://github.com/kbknapp/cargo-outdated); A cargo subcommand for displaying when Rust 
    dependencies are out of date.
  * [**cargo-sort-ck**](url:https://github.com/DevinR528/cargo-sort-ck); Checks that your `Cargo.toml` dependencies are 
    sorted alphabetically.
  * [**cargo-tarpaulin**](url:https://github.com/xd009642/tarpaulin); Code coverage tool for your Rust projects
  * [**cargo-tomlfmt**](url:https://github.com/tbrand/cargo-tomlfmt); Formatting `Cargo.toml`
* Debugging
  * [**cargo-profiler**](url:https://github.com/svenstaro/cargo-profiler); A cargo subcommand to profile your applications.
  * [[**cargo-]flamegraph**](https://github.com/flamegraph-rs/flamegraph); A Rust-powered flamegraph generator with 
    additional support for Cargo projects! It can be used to profile anything, not just Rust projects!
  * [**cargo-valgrind**](url:https://github.com/jfrimmel/cargo-valgrind); Runs a binary, example, bench, ... inside 
    valgrind to check for memory leaks. Helpful in FFI contexts.
* Useful
  * [**cargo-cache**](https://github.com/matthiaskrgr/cargo-cache); Display information on the cargo cache (`~~/.cargo/` 
    or `$CARGO_HOME`). Optional cache pruning.
  * [**cargo-deps**](https://github.com/m-cat/cargo-deps#instructions); Cargo subcommand for building dependency graphs of 
    Rust projects.

For typical builds I use `cargo make`; I have a Git pre-push hook that runs `cargo fmt`, `cargo clippy`, `cargo clean`, 
and `cargo build` to ensure clean pushes, especially when using CI.

## IntelliJ IDEA

Yes, I know it's not actually part of the toolchain, but it is really well integrated. While the support for cargo as a 
build tool is great, there are two preferences you need to set for each project.

Open the preferences dialog, (⌘-,) on a mac.

* Languages & Frameworks
  * Rust
    * Cargo
      * **Check** "Watch Cargo.toml"
      * **Check** "Compile all project targets if possible"
      * **Select** "Clippy" as the "External linter"
      * **Check** "Run external linter to analyze code on the fly"
    * Rustfmt
      * **Check** "Run rustfmt on Save"