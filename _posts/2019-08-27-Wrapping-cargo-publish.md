---
title: Wrapping cargo publish
layout: postx
category: code
tags: [rust, cargo]
---

[Rust](https://www.rust-lang.org/) is not only a really interesting and productive 
language, it also has a great toolchain with [Cargo](https://doc.rust-lang.org/cargo/index.html) 
and [rustup](https://rustup.rs/). I've been doing some personal wok in Rust for a 
little while, and as others warned me it took a while to get used to having such a 
picky compiler (like programming in Ada again!) but once you understand a little
more about what's going on it's pretty easy to make that thin happy.

So, having started a fun little project or two (see [crates.io](https://crates.io/users/johnstonskj))
I started getting a little frustrated with the Cargo publish step. The tool is great,
but orchestrating the steps to ensure the publish step is clean wasn't becoming
the habit it needed to be. I had to remember:

1. Make sure the build is **really** clean before you start.
2. Make sure the version number is correct and consistent between `Cargo.toml`,
   `README.me`, and any other files that may embed it.
3. Publish.
4. **IF** publish was successful, then tag the Git repo with the same version
   number from step 2.
   
So, I wrote a wrapper, it isn't exactly rocket surgery, but it saves me the headache
of realizing I tagged Git first but the publish failed and I need to do another
commit and reset the tags. Or, publishing, feeling good and realizing I didn't update 
the history in the readme file.

## Script

I simply named it `cargopub`, but here it is.

``` bash
#!/usr/bin/env bash

if [[ "$1" == "" ]] ; then
    echo "Error: a version message is required as first parameter"
    exit 1
fi

pushd $(git rev-parse --show-toplevel)

if ! cargo clean && cargo fmt -- --check && cargo test && cargo doc --no-deps ; then
    echo "Error: failed cargo pre-requisite steps"
    exit 2
fi

if [[ -e Cargo.toml ]] ; then
    VER=$(grep "^version = " Cargo.toml | egrep -o "[0-9\.]+(\-[a-zA-Z0-9\-_]+)?")
    echo "Info: about to publish version $VER"

    if [[ -e README.md ]] ; then
        if ! grep -q "\*\*$VER\*\*" README.md; then
            echo "Error: version not listed in README.md"
            exit 3
        fi
    fi

    if cargo publish; then
        git tag -a "v$VER" -m "$1"
        git push --tags --no-verify
        popd
    else 
        echo "Error: cargo publish command failed"
        exit 4
    fi
else
    echo "Error: no cargo file; are you in the right place?"
    exit 5
fi
```

