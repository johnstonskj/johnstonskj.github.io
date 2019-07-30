---
title: More on Rust and Travis
layout: postx
category: code
tags: [rust, travis, ci]
---

This is a follow on to the post I wrote earlier this month, {% link _posts/2019-07-19-Rust-and-Travis----Ouch.md %}.

As I am ~persistent~ stubborn, I continued to work on the problem. As I saw
it my solution had a couple of unfortunate conditions, namely:

* The Travis configuration file just felt too long.
* I stand by the fact that my for loops were OK as embedded scripting
  but maybe I could do better.
* Adding the `$CRATES` environment variable was an elegant solution,
  but how to keep that in sync with the Cargo workspace members?
* Deployment didn't work.

## New Travis configuration

```yaml
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
  - CARGO_FLAGS=--verbose
  - CARGO_LINTER=fmt
  - CARGO_DEPLOY=1
  - secure: "SwxX4DVDM7eS3AjJhdDCAIWIYi9IMfiYIAu+n4Fx4fkPf8RwCg/h1gA4RV6WNZBpus8fzD065GHnYWP6BVllyFzoF2WYG9xBTcSA+gJHFrGSSqa7xNLEZCTUkSb0KLtVv7GbgVZgH4lHgh+BTfAYwTn/ty9bBoz+qlF9k0tRJsP7q8KsL+XSZWIQdQuuCFnjnkaNYoJPg3Iz4BDV77Wyr2hIK1bsos1NNslCUJxp3d3O16lEd0L2uU19iZ6ORgOaTq5NUHZ/mTaRFmtLgZjHHNp0YYaoZCZjNqG5PDXZm0vU7e7rprx6s9p1O9Ll6jEAQcE07tNa1Om26hsYCS4HP5+zqNrhb6d5oB6n5NjobOK3E1AamgbnJglh96lxqreOICGRHEwYurgq+IyxXFE+xGfkxsi45xcdiX0HueFxYgfjbjtB5osXhE0pci2UXMDwcNhjqPs3WZ1cCPQKjYx64Rxt1bnOHLj8BqNmk075wDtHNqzbNI5k5VK9IvajNfXnVLTghFqIccSHNOswVRAmcnH+nc83bj9XCe4JMOh0UzIngg+wRVIcvrTkxWWdVAE0m/8XAJr06ZWF2bGRK2AATJqSeonyBpYxl7kK/qJoUs8oCrcw0KIVe1/Wd7HUwLoy4zxuVxH07F0H/v231Zip3XMvz8P3/1Sfm27MuMYuZlw="

matrix:
  # Performance tweak
  fast_finish: true
  # Ignore failures in nightly, not ideal, but necessary
  allow_failures:
  - rust: nightly

  # Only run the formatting check for stable, turn off deployments
  include:
  - name: 'Rust: linter check'
    rust: stable
    install:
    - ci/cargo-lint.sh --install
    env:
    - CARGO_DEPLOY=0
    script:
    - ci/cargo-lint.sh

# Script supports packages and workspaces
script:
- ci/cargo-build.sh

# Deployment script, this is under test
deploy:
  provider: script
  on:
    tags: true
    all_branches: true
    condition: "$TRAVIS_RUST_VERSION = stable && $TRAVIS_OS_NAME = linux && $CARGO_DEPLOY = 1"
  script: ci/cargo-publish.sh

# Only initiate build on master or tag branches
branches:
  only:
  - master
  - /\d+\.\d+(\.\d+)?(\-[a-z]+[a-zA-Z0-9]*)?/

# Suppress at least some emails
notifications:
  email:
    on_success: never
```

## CI Scripts

### Config

``` bash
if $(grep -q "^\[workspace\]$" Cargo.toml) ; then
    export CARGO_WORKSPACE=1
    #
    # This will extract the set of members from the workspace's Cargo.toml
    # file, it will construct a comma separated list in the $CRATES
    # environment variable, these SHOULD be listed in order of dependency
    # in your workspace so that publish steps can work effectively.
    #
    export CRATES=$(cat Cargo.toml \
        | egrep -o '"[^"]+"' \
        | tr '"'  ' ' \
        | tr '\n' ' ' \
        | tr -s '[:space:]' \
        | sed -e 's/^[[:space:]]*//' \
        | sed -e 's/[[:space:]]*$//' \
        | tr ' ' ',' \
    )
    echo "Cargo workspace contains ( $CRATES ) crates"
else
    export CARGO_WORKSPACE=0
fi
```

### Build

``` bash
source ci/cargo-config.sh

if [[ $CARGO_WORKSPACE = 1 ]] ; then
    WS_FLAGS="--all"
else
    WS_FLAGS=""
fi

if [[ "$1" == "--clean" ]] ; then
    echo "Cleaning up first..."
    cargo clean $CARGO_FLAGS --release --doc
fi

echo "Running build, test, doc..."
cargo build $CARGO_FLAGS $WS_FLAGS && \
cargo test $CARGO_FLAGS $WS_FLAGS && \
cargo doc $CARGO_FLAGS $WS_FLAGS --no-deps
```

### Lint-ing

``` bash
#!/usr/bin/env bash

source ci/cargo-config.sh

if [[ "$CARGO_LINTER" == "" ]] ; then
    echo >&2 "Warning: no CARGO_LINTER environment variable set, doing nothing now"
    exit 1
else
    if [[ "$1" == "--install" ]] ; then
        let "exit_code=0"
        for CMD in ${CARGO_LINTER//,/ }
        do
            case "$CMD" in
            fmt)
                rustup component add rustfmt
                let "exit_code += $?"
                ;;
            clippy)
                rustup component add clippy
                let "exit_code += $?"
                ;;
            *)
                echo >&2 "Warning: unknown command $CMD"
                let "exit_code += 100"
                ;;
            esac
        done
        exit $exit_code
    else
        let "exit_code=0"
        for CMD in ${CARGO_LINTER//,/ }
        do
            case "$CMD" in
            fmt)
                ci/cargo-command.sh fmt --check $*
                let "exit_code += $?"
                ;;
            clippy)
                ci/cargo-command.sh clippy -D warnings $*
                let "exit_code += $?"
                ;;
            *)
                echo >&2 "Warning: unknown command $CMD"
                let "exit_code += 100"
                ;;
            esac
        done
        exit $exit_code
    fi
fi
```

### Run a command

``` bash
source ci/cargo-config.sh

if [[ $CARGO_WORKSPACE = 1 ]] ; then
    WS_FLAGS="--all"
else
    WS_FLAGS=""
fi

CARGO_COMMAND=$1
shift

if [[ $# = 0 ]] ; then
    cargo $CARGO_COMMAND $CARGO_FLAGS $WS_FLAGS
else
    cargo $CARGO_COMMAND $CARGO_FLAGS $WS_FLAGS -- $*
fi
```

### Deploy/Publish

``` bash
source ci/cargo-config.sh

if [[ "$CARGO_DEPLOY" = "0" ]] ; then
    # Just in case.
    echo "Skipping deployment step for now"
    exit 0
fi

if [[ "$CARGO_TOKEN" = "" ]] ; then
    # Ensure this is set as a global environment
    #  variable, *and* as a secure one.
    echo "Error: no CARGO_TOKEN environment variable"
    exit 2
fi

# There is no '--all' option on publish :-(
if [[ $CARGO_WORKSPACE = 1 ]] ; then
    # Refresh the Cargo.lock file first
    cargo update
    for CRATE in ${CRATES//,/ }
    do
        # must use locked otherwise it doesn't
        # correctly resolve local versions.
        cargo publish $CARGO_FLAGS --locked --token $CARGO_TOKEN --manifest-path $CRATE/Cargo.toml
    done
else
    cargo publish $CARGO_FLAGS --token $CARGO_TOKEN
fi
```

