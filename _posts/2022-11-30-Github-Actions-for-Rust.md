---
title: "Github Actions for Rust"
layout: postx
category: code
tags: [rust, pipeline]
---

[Github Actions](https://github.com/features/actions) have been a great
advantage since they were added, and have only grown in value as the number of
good quality building blocks increases. At the same time, Rust has a great
toolchain and I know I copied some example Rust action and with a little
editing I had the following.

```rust
name: Rust

on: [push]

jobs:
  build:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]

    runs-on: {% raw %}${{ matrix.os }}{% endraw %}

    steps:
      - uses: actions/checkout@v1

      - name: Install dependencies
        run: rustup component add rustfmt

      - name: Format
        run: cargo fmt -- --check

      - name: Build
        run: cargo build --verbose

      - name: Run tests
        run: cargo test --all-features --verbose

      - name: Docs
        run: cargo doc --no-deps
```

Fantastic, so I copied that into a bunch of projects and was pretty happy. But
felt like there was some complexity missing, for a start what tests am I
running, am I running benchmarks and/or examples? Do I run coverage at the
same time or separately from my main tests? I spent some time on a couple of
projects evolving my Rust workflow (which I still call pipelines), using jobs,
dependencies, and some inputs/outputs between jobs to get too this:


![Action Screenshot](/assets/img/posts/rust_workflow.png)

This new workflow is clearly a lot more complex but it has some nice
separation of concerns and allows quite a bit of parallelism. My test matrix
for a start has become a lot more interesting, I now combine not just the
platforms, but toolchain channel and test features. This last addition was
useful in finding a stupid bug that only showed up with `--no-default-features`.

```yaml
matrix:
  os: ["ubuntu-latest", "windows-latest", "macos-latest"]
  rust: ["stable", "beta", "nightly"]
  test-features: ["", "--all-features", "--no-default-features"]
```

Another feature that took a little time to figure out was the turning on/off
the execution of examples and benchmarks. I want them to run if the
corresponding directory exists, but don't want to have all manner of
conditionals in scripts that fake success if there are no tests. So, I added
the following simple job which generates a flag for each directory and can
then gate subsequent tests based on these flags. Specifically the two
downstream jobs have the following: `if:
needs.check_tests.outputs.has_benchmarks` or `has_examples`.

```yaml
check_tests:

  name: Check for test types
  runs-on: ubuntu-latest
  
  outputs:
    has_benchmarks: {% raw %}${{ steps.check_benchmarks.outputs.has_benchmarks }}{% endraw %}
    has_examples: {% raw %}${{ steps.check_examples.outputs.has_examples }}{% endraw %}
    
  steps:
    - name: Check for benchmarks
      id: check_benchmarks
      run: test -d benchmarks && echo "has_benchmarks=1" || echo "has_benchmarks=" >> $GITHUB_OUTPUT
      shell: bash
      
    - name: Check for examples
      id: check_examples
      run: test -d examples && echo "has_examples=1" || echo "has_examples=" >> $GITHUB_OUTPUT
      shell: bash
```

I'm sure there will be more iterations, but this was a pretty good step
forward and it will be the new baseline to add to my project template and
slowly add back into existing projects.

You can find this workflow at
[rust.tml](https://github.com/johnstonskj/rust-codes/blob/main/.github/workflows/rust.yml),
along with my
[security-audit.yml](https://github.com/johnstonskj/rust-codes/blob/main/.github/workflows/security-audit.yml)
(which includes a schedule to be notified of issues), and
[release.yml](https://github.com/johnstonskj/rust-codes/blob/main/.github/workflows/release.yml)
workflows. I also use the cool
[typos.yml](https://github.com/johnstonskj/rust-codes/blob/main/.github/workflows/typos.yml)
workflow on pull requests. 
