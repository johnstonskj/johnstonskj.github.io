---
title: Rust and Travis -- Ouch
layout: post
category: code
tags: [rust, travis, ci]
---

Having been using Rust a little at work for small tools and experimentation I
started on a set of crates for fun. My goal is to build a pretty simple, but 
usable terminal stock/portfolio tool, focusing right now on the 
[financial](https://github.com/johnstonskj/rust-financial) model and APIs.

The next step, after a bunch of initial exploration, development, and GitHub
versioning was to get a Continuous Integration (CI) build process in place. 
I have used [Travis CI](https://travis-ci.org/johnstonskj) for a number of 
projects already and found it pretty easy for most languages so that's where 
I started.

## Getting Started

The Rust book has a section on [Continuous Integration](https://doc.rust-lang.org/cargo/guide/continuous-integration.html#travis-ci)
which seemed like a great place to start, and specifically it included 
the following guidance on creating a `travis.yml` file.


```yaml
language: rust
rust:
  - stable
  - beta
  - nightly
matrix:
  allow_failures:
    - rust: nightly
```

This seemed to work, although it isn't terribly exciting. I clicked through
to read the Travis guidance on [Building a Rust Project](https://docs.travis-ci.com/user/languages/rust/)
next. This page had some additional topics that seemed like they'd be useful;

* should add `fast_finish: true` to the matrix to make builds faster
* should add `cache: cargo` to speed up builds as well
* if you're using cargo workspaces (and I am), explicitly add a `script`
  to pass `--all` into build and test commands.

The result of this guidance is the following build file, with two additional
pieces added from my existing build scripts.

``` yaml
language: rust
sudo: false # <-- always goodness
cache: cargo
rust:
  - stable
  - beta
  - 1.34.0 # <-- my minimal version
  - nightly
matrix:
  allow_failures:
    - rust: nightly
  fast_finish: true
script:
  - cargo build --verbose --all
  - cargo test --verbose --all
notifications: # <-- I always add this
  email:
    on_success: never
```

So, now I get a good spread of channels, I am happy to let `nightly` fail,
and I got the performance tweaks in there as well. 

## Getting Frustrated

However, there are more things I want to do, for example:

1. Test build on Linux, macOS, and Windows
1. Build documentation
1. Include [rustfmt](https://github.com/rust-lang/rustfmt) checking
1. Include [clippy](https://github.com/rust-lang/rust-clippy) lint checking
1. Include [Coveralls](https://coveralls.io/) integration
1. Publish the included crates to [crates.io](https://crates.io/)

Some of this was more work that I had hoped.

Getting operating system support was pretty easy, I just added the following 
right below my `rust` section.

``` yaml
os:
- linux
- osx
- windows
```

So, now I get a lot more builds running, but unfortunately I never managed to
get a successful one on Windows. However, as Travis is careful to point out that 
Windows support is at best a beta right now I simply removed it from the list
until I care enough to figure out the issue.

As for documentation, because this repository is a cargo workspace I cannot 
simply call `cargo doc`, and `--all` doesn't work. So, I have to add the 
following three lines to the `script` list.

``` yaml
- cargo doc   --verbose --package fin_model --no-deps
- cargo doc   --verbose --package fin_data --no-deps
- cargo doc   --verbose --package fin_iex --no-deps
```

This seemed ugly, and if ever I add a crate or remove one I have to manage 
separate lines (also see deployment below). I want to be able to specify 
the set of crates in one place and then just loop over them. Now, this seems
like a reason to write an external script, but I this is a case where I am
happy to include the bash logic in the yaml file, it's simple and clear what's
going on. 

First, add a new global environment variable, as a comma separated list of
crate names

``` yaml
  global:
  - CRATES=fin_model,fin_data,fin_iex
```

Then replace the three previous script lines with a bash for loop.

``` yaml
- |
  for CRATE in ${CRATES//,/ }
  do
      cargo doc   --verbose --package $CRATE --no-deps
  done
```

As for rustfmt, clippy, and Coveralls I found a number of repos in GitHub
with build integrations but they all seemed to have horrible scripts, and
managed complex dependencies. One goal I tried to set myself was to see how 
much of the remainder of the build I could define *without* resorting to 
external scripts or ugly inline scripts.

I also decided that rather than run these three tools
all the time for all the matrix of tests I would simply run them once, using
the stable channel. To accomplish this I added an `include` section under
the build `matrix` and defined three sections, one for each task.

Running rustfmt turned out to be pretty straightforward, and adding an `install`
step to add the command and a single-line script to run it.

``` yaml
  - name: 'Rust: format check'
    rust: stable
    install:
    - rustup component add rustfmt
    script:
    - cargo fmt --verbose --all -- --check
```

Running clippy was pretty much the same as rustfmt. One problem still persists
with clippy in that it has an occasional build error, code that builds fine, 
tests fine, documents fine, throws version mismatch errors when clippy tries.
So, to allow testing with or without it I have put in place a conditional so 
that it only ever enables as a step if the environment variable `ENABLE_CLIPPY`
is set to 1.

``` yaml
  - name: 'Rust: style check'
    if: env(ENABLE_CLIPPY) = 1
    rust: stable
    install:
    - rustup component add clippy
    script:
    - cargo clippy --verbose --all -- -D warnings
```

Code coverage is similar, although no install step was required.

``` yaml
  - name: 'Rust: code coverage'
    rust: stable
    os: linux
    script:
    - cargo tarpaulin --verbose --ciserver travis-ci --coveralls $TRAVIS_JOB_ID
```

The deployment stage hasn't yet been tested, but I've copied in the best advice
so far, and made it conditional only branch, tags, and environment.

``` yaml
deploy:
  provider: cargo
  token:
    secure: GlZuK.....ZND5s=
  on:
    tags: true
    branch: master
    condition: "$TRAVIS_RUST_VERSION = stable && $TRAVIS_OS_NAME = linux"
```

## Getting It Right

So, here is my resulting
[`.travis.yml`](https://github.com/johnstonskj/rust-financial/blob/master/.travis.yml)
file in all it's (mostly working) glory! I haven't tested the deployment stuff
yet, and the tarpaulin step doesn't seem to work, but it's close enough for now.

``` yaml
# Common language header
language: rust
sudo: false
cache: cargo

# Channels and versions I want to build
rust:
- stable
- beta
- 1.34.0
- nightly

# Operating systems I want to test
os:
- linux
- osx

# Set global environment only
env:
  global:
  - RUST_BACKTRACE=1
  - CRATES=fin_model,fin_data,fin_iex

matrix:
  # Performance tweak
  fast_finish: true
  # Ignore failures in nightly, not ideal, but necessary
  allow_failures:
  - rust: nightly

  # Only run the formatting check for stable
  include:
  - name: 'Rust: format check'
    rust: stable
    install:
    - rustup component add rustfmt
    script:
    - cargo fmt --verbose --all -- --check

    # Only run the style check for stable, if enabled
  - name: 'Rust: style check'
    if: env(ENABLE_CLIPPY) = 1
    rust: stable
    install:
    - rustup component add clippy
    script:
    - cargo clippy --verbose --all -- -D warnings

    # Only run code coverage for stable
  - name: 'Rust: code coverage'
    rust: stable
    os: linux
    script:
    - cargo tarpaulin --verbose --ciserver travis-ci --coveralls $TRAVIS_JOB_ID

# Custom script
# * adding '--all' for workspaces on build/test
# * adding '--package' for workspaces on doc
script:
- cargo build --verbose --all
- cargo test  --verbose --all
- |
  for CRATE in ${CRATES//,/ }
  do
      cargo doc   --verbose --package $CRATE --no-deps
  done

# Cargo/Crates integration
deploy:
  provider: cargo
  token:
    secure: GlZuK.....ZND5s=
  on:
    tags: true
    branch: master
    condition: "$TRAVIS_RUST_VERSION = stable && $TRAVIS_OS_NAME = linux"

# Only initiate build on mainline branches
branches:
  only: master

# Suppress at least some emails
notifications:
  email:
    on_success: never
```
