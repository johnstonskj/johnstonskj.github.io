---
title: More on Rust and Travis
layout: postx
category: code
tags: [rust, travis, ci]
---

This is a follow on to the post I wrote earlier this month, {% link _posts/2019-07-19-Rust-and-Travis----Ouch.md %}.

As I am ~~persistent~~ stubborn, I continued to work on the problem. As I 
saw it my solution had a couple of unfortunate conditions, namely:

* The Travis configuration file just felt too long.
* I stand by the fact that my for loops were OK as embedded scripting
  but maybe I could do better.
* Adding the `$CRATES` environment variable was an elegant solution,
  but how to keep that in sync with the Cargo workspace members?
* Deployment didn't work.

## New Travis configuration

So, below is the new [`.travis.yml`](https://github.com/johnstonskj/rust-financial/blob/master/.travis.yml)
for the workspace. This removes all scripting from the configuration 
and uses a series of global flags that control the behavior of the CI 
scripts. Note the `install` list that includes a Git checkout and a 
script call. I have moved all of the scripts from `rust-financial` and
moved to [`rustt-ci`](https://github.com/johnstonskj/rust-ci).

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
  - CARGO_BIN=ci/bin
  - CARGO_DEBUG=1
  - CARGO_DEPLOY=1
  - CARGO_FLAGS=--verbose
  - CARGO_LINTER=fmt
  - secure: "SwxX4DVDM7eS3AjJhdDCAIWIYi9IMfiYIAu+n4Fx4fkPf8RwCg/h1gA4RV6WNZBpus8fzD065GHnYWP6BVllyFzoF2WYG9xBTcSA+gJHFrGSSqa7xNLEZCTUkSb0KLtVv7GbgVZgH4lHgh+BTfAYwTn/ty9bBoz+qlF9k0tRJsP7q8KsL+XSZWIQdQuuCFnjnkaNYoJPg3Iz4BDV77Wyr2hIK1bsos1NNslCUJxp3d3O16lEd0L2uU19iZ6ORgOaTq5NUHZ/mTaRFmtLgZjHHNp0YYaoZCZjNqG5PDXZm0vU7e7rprx6s9p1O9Ll6jEAQcE07tNa1Om26hsYCS4HP5+zqNrhb6d5oB6n5NjobOK3E1AamgbnJglh96lxqreOICGRHEwYurgq+IyxXFE+xGfkxsi45xcdiX0HueFxYgfjbjtB5osXhE0pci2UXMDwcNhjqPs3WZ1cCPQKjYx64Rxt1bnOHLj8BqNmk075wDtHNqzbNI5k5VK9IvajNfXnVLTghFqIccSHNOswVRAmcnH+nc83bj9XCe4JMOh0UzIngg+wRVIcvrTkxWWdVAE0m/8XAJr06ZWF2bGRK2AATJqSeonyBpYxl7kK/qJoUs8oCrcw0KIVe1/Wd7HUwLoy4zxuVxH07F0H/v231Zip3XMvz8P3/1Sfm27MuMYuZlw="

install:
- git clone https://github.com/johnstonskj/rust-ci.git ci
- ci/bin/cargo-lint.sh --install

matrix:
  # Performance tweak
  fast_finish: true
  # Ignore failures in nightly, not ideal, but necessary
  allow_failures:
  - rust: nightly

# Script supports packages and workspaces
script:
- ci/bin/cargo-build.sh
- ci/bin/cargo-lint.sh

# Deployment script, this is under test
deploy:
  provider: script
  on:
    tags: true
    all_branches: true
    condition: "$TRAVIS_RUST_VERSION = stable && $TRAVIS_OS_NAME = linux && $CARGO_DEPLOY = 1"
  script: ci/bin/cargo-publish.sh

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

The following are the individual components called from the configuration.

### Config

This script is not called from the configuration, rather is is sourced by
each of the following scripts to set up the environment. Firstly, it determines
whether this is a workspace or crate repository. Then, if it is a workspace,
it creates the `$CRATES` environment variable directly from the members
listed in the `Cargo.toml` file.

``` bash
#!/usr/bin/env bash

if [[ "$CARGO_BIN" = "" ]] ; then
    CARGO_BIN=$(dirname "$0")
fi

source $CARGO_BIN/logging.sh

debug "running cargo CI commands from $CARGO_BIN"

if [[ ! -f "Cargo.toml" ]] ; then
    fatal "no Cargo.toml file, are you running in your project root?" 2>&1
fi

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
    info "cargo workspace contains ( $CRATES ) crates"
else
    export CARGO_WORKSPACE=0
fi

if [[ $CARGO_DEBUG = 1 ]] ; then
    debug "setting debug flags (CARGO_FLAGS, RUST_BACKTRACE, RUST_LOG)"
    RUST_BACKTRACE=1
    RUST_LOG=info
    if [[ ! $CARGO_FLAGS = *verbose* ]] ; then
	    CARGO_FLAGS="$CARGO_FLAGS --verbose"
    fi
fi
```

### Build

This is the build script, it determines the correct flags to use depending
on whether this ia a workspace build (from `$CARGO_WORKSPACE`). It performs
the build, test, and doc tasks into a single script.

``` bash
#!/usr/bin/env bash

source $CARGO_BIN/cargo-config.sh

if [[ $CARGO_WORKSPACE = 1 ]] ; then
    WS_FLAGS="--all"
else
    WS_FLAGS=""
fi

if [[ "$1" == "--clean" ]] ; then
    info "cleaning up first..."
    cargo clean $CARGO_FLAGS --release --doc
fi

info "running build, test, doc..."
cargo build $CARGO_FLAGS $WS_FLAGS && \
cargo test $CARGO_FLAGS $WS_FLAGS && \
cargo doc $CARGO_FLAGS $WS_FLAGS --no-deps
```

### Lint-ing

This performs any lint-like tasks in a single script. Currently it supports
`rustfmt` and `clippy` only. It assumes that the environment variable
`$CARGO_LINTER` contains a comma separated list of commands that you wish
to run. This may be run for all builds, in which case add it as a line in
the top-level `script` list, or as I have done, create a new build in
the matrix.

``` bash
#!/usr/bin/env bash

source $CARGO_BIN/cargo-config.sh

if [[ "$CARGO_LINTER" == "" ]] ; then
    warning "no CARGO_LINTER environment variable set, doing nothing now"
    exit 1
else
    if [[ "$1" == "--install" ]] ; then
        info "installing lint-like tools"
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
                warning "unknown command $CMD"
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
                $CARGO_BIN/cargo-command.sh fmt --check $*
                let "exit_code += $?"
                ;;
            clippy)
                $CARGO_BIN/cargo-command.sh clippy -D warnings $*
                let "exit_code += $?"
                ;;
            *)
                warning "unknown command $CMD"
                let "exit_code += 100"
                ;;
            esac
        done
        exit $exit_code
    fi
fi
```

### Run a command

This script isn't currently called from the configuration file directly,
although it could. It runs an arbitrary Cargo command taking into account
any required flags, and passing any flags to the command.

``` bash
#!/usr/bin/env bash

source $CARGO_BIN/cargo-config.sh

if [[ $# -lt 1 ]] ; then
    fatal "no CARGO_COMMAND argument supplied"
fi

if [[ $CARGO_WORKSPACE = 1 ]] ; then
    WS_FLAGS="--all"
else
    WS_FLAGS=""
fi

CARGO_COMMAND=$1
shift

debug "running $CARGO_COMMAND"

if [[ $# = 0 ]] ; then
    cargo $CARGO_COMMAND $CARGO_FLAGS $WS_FLAGS
else
    cargo $CARGO_COMMAND $CARGO_FLAGS $WS_FLAGS -- $*
fi
```

### Deploy/Publish

This script is an ongoing, and painful, attempt at getting Travis to publish the
crates in the workspace. Currently, it has worked for a random single crate in 
the workspace, but not all. Still, that's progress.

``` bash
#!/usr/bin/env bash

source $CARGO_BIN/cargo-config.sh

if [[ "$CARGO_DEPLOY" = "0" ]] ; then
    # Just in case.
    info "skipping deployment step for now"
    exit 0
fi

if [[ "$CARGO_TOKEN" = "" ]] ; then
    # Ensure this is set as a global environment
    #  variable, *and* as a secure one.
    error "no CARGO_TOKEN environment variable"
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

Note that in the Travis configuration I have switched from the `cargo` 
provider and simply use `script`. This allows me to move all the logic 
into the script above. Also note the new condition using `$CARGO_DEPLOY`
to gate deployment. 

``` yaml
deploy:
  provider: script
  on:
    tags: true
    all_branches: true
    condition: "$TRAVIS_RUST_VERSION = stable && $TRAVIS_OS_NAME = linux && $CARGO_DEPLOY = 1"
  script: ci/cargo-publish.sh

```

Finally, this uses `all_braches: true` but still `tags: true` to only 
use tagged builds. **BUT** No builds occured when I tagged a build. It
turns out that Travis uses the tag name as the branch name when it runs
the build and so you have to include something in the top level
`branches` block to whitelist those builds. I used a reasonably safe 
regex for semantic version tags.

``` yaml
branches:
  only:
  - master
  - /\d+\.\d+(\.\d+)?(\-[a-z]+[a-zA-Z0-9]*)?/
```

