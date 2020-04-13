---
title: Pattern Matching (Rustioms)
layout: postx
categories: [code]
tags: [rust, match, idioms]
---

A lot of languages have pattern matching, so not all of this is Rust magic, but the set of things you can do with 
pattern matching, and the powerful alternatives it also provides, help write both efficient and readable code. This is 
definitely a topic I will probably come back to as I keep discovering new and better ways to work with the tools that
Rust provides.

I included an example of the Rust `match` expression in a previous post, copied below.

```rust
impl Display for MyError {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        write!(
            f,
            "{}",
            match self {
                MyError::NotFound => "Where'd it go?".to_string(),
                MyError::DidntLikeThis(s) =>
                    format!("This looks bad: '{}'", s),
                MyError::IOError(err) =>
                    format!("Well crap. '{}'", err),
            }
        )
    }
}
```

In this example the `match` looks very much like a case or switch statement, matching simply against static identifiers. 
The real fun comes when you step beyond just "id" patterns.

For example, imagine I have two `Option` values that interact in some way, so rather than writing nested matches we can make use 
of the Rust tuple type and construct a tuple with these two values and then match on particular tuple values that 
further deconstruct their components. Tuples are nice because they are not constrained to use values of the same time, 
so I could pass flags in as tuple values to differ the handling of the left/right values as well.

```rust
match (left, right) {
    (Some(v), None) => do_this(v),
    (None, Some(v)) => do_that(v),
    _ => (),
}
```

Now, is often the case in Rust there is an even easier way to accomplish this specific case as it turns out. `Option` 
has an `xor()` function, which is really what we are doing in the match above, so we can use `if let` to unpack the 
value from xor, if it is present.

```rust
if let Some(v) = left.xor(right) {
    do_that(v)
}
```

The `if let` construct is also a useful alternative where you care only about one specific pattern, especially if you have
no "else" behavior. A personal rule-of-thumb, if you have a single case, no "else", use `if let`, if you have multiple 
cases, regardless of whether there's an "else", use `match`, the _one-case-and-else_ could be either `if let` or `match` 
and the choice for me tends to be on the style of the surrounding code.

Another example is the explicit optional processing, in the following there may or may not be a second value to deal 
with, and the use of if let is actually a good choice because it's very explicit that I really don't care about the 
`None` case at all, I either process `and_maybe_this` or just pass on by.

```rust
fn optionally_do_this<T>(to_this: T, and_maybe_this: Option<T>) {
    do_this(to_this);
    if let Some(as_well) = and_maybe_this {
        do_this(as_well);
    }
}
```

Another easy problem, one that happens often when you start writing, is the overly nested match, logically it's 
perfectly OK, and makes sense, but there's something just off about the result. One blog post called this the 
_Russian doll_ effect, things inside things.

```rust
match start_with {
    None => {}
    Some(next) => match process_one(next) {
        None => {}
        Some(next) => match process_two(next) {
            None => {}
            Some(next) => {
                process_three(next);
            }
        },
    },
}
```

Well, it turns out `Option` itself can solve this. The `and_then()` function will apply a function to a value if that 
value is `Some(_)`, and return an `Option` so you can chain them. So, the code above can be written as just a chain, 
and is way more readable.

```rust
start_with
    .and_then(process_one)
    .and_then(process_two)
    .and_then(process_three);
```
