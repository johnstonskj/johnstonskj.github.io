---
title: Test and Debug (Rustioms)
layout: postx
category: code
tags: [rust, test, debug, log, idioms]
---

### White-box test modules

[Rust Fuzz Book](https://rust-fuzz.github.io/book/introduction.html)
[proptest](https://altsysrq.github.io/proptest-book/intro.html)

[quickcheck](https://github.com/BurntSushi/quickcheck)

### Black box tests

```toml
[dev-dependencies]
test-generator = "0.3.0"
```

```rust
#![cfg(test)]
extern crate test_generator;

use std::fs;
use std::path::PathBuf;
use test_generator::test_resources;

#[test_resources("tests/data/good/*.json")]
fn verify_good_examples(resource: &str) {
    println!("verify_good_examples reading file {}", resource);
    // test
    assert!(false);
}
```

## Examples

## Documentation

## Benchmarks

```toml
[dev-dependencies]
criterion = "0.3"
```


```rust
use criterion::{criterion_group, criterion_main, Criterion, Throughput};

pub fn benchmark_time(c: &mut Criterion) {
    let mut group = c.benchmark_group("performance");
    group.measurement_time(Duration::new(20, 0));

    group.bench_function("GET /v0/schema/human+biometrics", |b| {
        b.iter(|| call_get_schema())
    });
    group.bench_function("GET /v0/schema/human+biometrics unmodified", |b| {
        b.iter(|| call_get_schema_unmodified())
    });

    group.finish();
}

pub fn benchmark_throughput(c: &mut Criterion) {
    let mut group = c.benchmark_group("throughput-bytes");
    group.measurement_time(Duration::new(20, 0));

    // We need to assume this wont change during execution of this test.
    group.throughput(Throughput::Bytes(call_get_schema() as u64));

    group.bench_function("GET /v0/schema/human+biometrics", |b| {
        b.iter(|| call_get_schema())
    });

    group.finish();
}

criterion_group!(benches, benchmark_time, benchmark_throughput);
criterion_main!(benches);
```

## Documentation Links

