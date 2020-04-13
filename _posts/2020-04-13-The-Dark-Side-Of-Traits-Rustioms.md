---
title: The Dark Side of Traits (Rustioms)
layout: post
categories: [code]
tags: [rust, traits, idioms]
---

In [_It's Traits (Almost) All the Way Down_]({% post_url 2020-04-10-Its-Traits-All-The-Way-Down-Rustioms %}), I covered 
some of the really cool aspects of traits in Rust, and some of the ways they are used in the core. Unfortunately Rust 
traits have some rather idiosyncratic behaviors that can make them frustrating at first. None of the frustrations below
are really blockers, well they may be the first time you encounter them, and they are part of how the language implements
traits in a performant and safe manner. 

## Returning a Trait

Consider the following function, I want to return some value that implements the `Display` trait. That seems like it
should be pretty straightforward, right? If you've worked with other languages that support traits or interfaces you
just treat the trait name as a type name.

```rust
use std::fmt::Display;

pub fn this_is_what_i_want_to_write() -> Display {
    todo!()
}
```

Well, kinda, except that the compiler _may_, and Clippy _will_, give you a message that it's considered bad form to use a 
trait in this way without the keyword `dyn`.

So, how about this?

```rust
use std::fmt::Display;

pub fn illegal_1() -> dyn Display {
    todo!()
}
```

Ah, no. It turns out that we can't do this because the size of a _trait object_ is undefined and we can't put an unknown 
sized value (concrete types implement another specific marker trait [`Sized`](https://doc.rust-lang.org/std/marker/trait.Sized.html)) 
on the stack to return it.

Now, we might know about `std::marker::Sized`, and try using trait bounds, so now we have explicitly said that the 
thing we are going to return is not just a `Display` but also sized!

```rust
use std::fmt::Display;
use std::marker::Sized;

pub fn illegal_2<T>() -> T
where
    T: Display + Sized,
{
    let value: u16 = 10;
    value
}
```

This now fails because we cannot tell the compiler to coerce the `u16` value into `(dyn Display + Sized)`. Dammit. 
So, do references work any better?

```rust
use std::fmt::Display;

pub fn illegal_3() -> &'static dyn Display {
    let value: u16 = 10;
    &value
}
```

Nope, we fail the borrow checker on the last line as we cannot return a reference to `value` as it's on the stack and 
therefore gets dropped as we exit the function. As an aside, the following call is legal because we are passing back
a reference but a reference only to the thing we started with in the first place.

```rust
fn legal_pass_through<T: Display>(v: &T) -> &dyn Display {
    // do something
    v
}
```

Here's where a handy tool be the name of [`Box`](https://doc.rust-lang.org/std/boxed/struct.Box.html) comes in. The 
following works because the new `Box` value is on the heap so we can return it, it uses a trait bound as a parameter type 
(not return type) and so it generally gives us less pain.

```rust
use std::fmt::Display;
use std::boxed::Box;

pub fn legal() -> Box<dyn Display> {
    let value: u16 = 10;
    Box::new(value)
}
```

There are other container types in Rust, but we'll save that for another post.

So, bottom line, if you want to return a trait object (some value only known by one or more traits it implements) you 
can use a reference if it's safe to do so, or use `Box` or one of it's friends.


## Traits and Function Pointers

TBD

### Functions as Arguments

```rust
pub fn wrapper_1(s: String, callback: fn(String) -> bool) -> bool {
    callback(s)
}

pub fn wrapper_2a(s: String, callback: impl Fn(String) -> bool) -> bool {
    callback(s)
}

pub fn wrapper_2b<F>(s: String, callback: F) -> bool
where
    F: Fn(String) -> bool, // "impl" is not allowed here
{
    callback(s)
}

pub fn wrapper_3(s: String, callback: &dyn Fn(String) -> bool) -> bool {
    callback(s)
}

pub type Callback = fn(String) -> bool;
pub type CallbackTwo = &'static dyn Fn(String) -> bool;
// pub type CallbackThree = impl Fn(String) -> bool; // `impl Trait` in type aliases is unstable

pub fn wrapper_4(s: String, callback: Callback) -> bool {
    callback(s)
}

pub fn wrapper_5(s: String, callback: Callback) -> bool {
    callback(s)
}
```

### Functions as Return Values

```rust
// pub fn make_starts_with_1(prefix: &str) -> fn(String) -> bool {
//                                            ------------------ expected `fn(std::string::String) -> bool` because of return type
//     let prefix = prefix.to_string();
//     move |s: String| s == prefix
//     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected fn pointer, found closure
// }

pub fn make_starts_with_2a(prefix: &str) -> impl Fn(String) -> bool {
    let prefix = prefix.to_string();
    move |s: String| s == prefix
}

// pub fn make_starts_with_2b<F>(prefix: &str) -> F
//                            -                   - expected `F` because of return type
//                            |
//                            this type parameter
// where
//     F: Fn(String) -> bool,
// {
//     let prefix = prefix.to_string();
//     move |s: String| s == prefix
//     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected type parameter `F`, found closure
// }

// pub fn make_starts_with_3(prefix: &str) -> &dyn Fn(String) -> bool {
//     let prefix = prefix.to_string();
//     &move |s: String| s == prefix
//     ^----------------------------
//     ||
//     |temporary value created here
//     returns a reference to data owned by the current function
// }

// pub fn make_starts_with_4(prefix: &str) -> Callback {
//                                            ------------------ expected `fn(std::string::String) -> bool` because of return type
//     let prefix = prefix.to_string();
//     move |s: String| s == prefix
//     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected fn pointer, found closure
// }

// pub fn make_starts_with_4(prefix: &str) -> CallbackTwo {
//     let prefix = prefix.to_string();
//     &move |s: String| s == prefix
//     ^----------------------------
//     ||
//     |temporary value created here
//     returns a reference to data owned by the current function
// }
```

## Documentation Links

* [`std::boxed::Box`](https://doc.rust-lang.org/std/boxed/struct.Box.html); A pointer type for heap allocation.
* [`std::marker::Sized`](https://doc.rust-lang.org/std/marker/trait.Sized.html); Types with a constant size known at 
  compile time.