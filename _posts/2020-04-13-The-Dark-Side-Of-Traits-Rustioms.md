---
title: The Dark Side of Traits (Rustioms)
layout: postx
category: code
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

At some point I'll write some notes on Rust support for functional programming especially the ubiquity of iterators, a 
discussion that will require us to look at function pointers or closures (AKA function objects, lambdas, etc.). To 
easily pass functions or closures around it's necessary to understand the primitive type 
[`fn`](https://doc.rust-lang.org/std/primitive.fn.html) and the traits 
[`Fn`](https://doc.rust-lang.org/std/ops/trait.Fn.html), [`FnOnce`](https://doc.rust-lang.org/std/ops/trait.FnMut.html), 
and [`FnMut`](https://doc.rust-lang.org/std/ops/trait.FnOnce.html). The distinction is subtle, but where we saw the 
keyword `fn` used to define a new function this was a _thing_ whose type is the primitive type `fn`. This function 
implements one or more of the function traits as do closures which we see a lot when we use iterators. 

So, consider the following example that filters a list of strings to those that start with the value `"hello"`. In
`filter_strings_1` we pass a function pointer to `filter`, but that function has to hard code the value to test against
which doesn't seem efficient. In `filter_strings_2` we pass a closure to `filter` which is creating a function-like
object that implements some of the function traits so that it may be called in the same way as our actual function.


```rust
fn string_starts_with_hello(s: &String) -> bool {
    s.starts_with("hello")
}

fn filter_strings_1(ss: Vec<String>) -> Vec<String> {
    ss.into_iter().filter(string_starts_with_hello).collect()
}

fn filter_strings_2(ss: Vec<String>) -> Vec<String> {
    ss.into_iter().filter(|s| s.starts_with("hello")).collect()
}
```

Now let's look at where it gets kinda complicated. I am creating a function named `wrapper` that takes a string and a 
callback function and after I've done _something_ to the string I'll call the callback with a new string. So, the 
first question is how to express the type of the callback parameter. It turns out there's actually quite a bit of 
choice as shown in the examples below.


### Functions as Arguments

```rust
pub fn wrapper_1(s: String, callback: fn(String) -> bool) -> bool {
    callback(s)
}

pub fn wrapper_2a<F>(s: String, callback: F) -> bool
where
    F: Fn(String) -> bool, // "impl" is not allowed here
{
    callback(s)
}

pub fn wrapper_2b(s: String, callback: impl Fn(String) -> bool) -> bool {
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

* `wrapper_1`; our parameter is defined as a function pointer using the primitive type `fn` followed by arguments and
  return type. 
* `wrapper_2a`; uses a trait bound to define the generic type `T` as a implementing the `Fn` trait. 
* `wrapper_2b`; uses a new syntax that has a similar meaning int that `callback` implements `Fn`, but this is a lot more
  compact and easier to read. Note that the `impl Trait` syntax isn't allowed everywhere, for example you cannot use it
  in there `where` clause in `wrapper_2a`.
* `wrapper_3`; uses a reference type, pretty simple.
* `wrapper_4` and  `wrapper_5`; these show how the approaches in `wrapper_1` and `wrapper_3` can be turned into type
   names, unfortunately you can't create a type for `dyn Fn()`, only a reference and `impl` is not legal in a type.
   
In general the standard library uses the trait bounds approach as in wrapper_2a, for example the following is the 
signature of `map` on `std::iter::Iterator`.

```rust
fn map<B, F>(self, f: F) -> Map<Self, F>
where
    F: FnMut(Self::Item) -> B, 
```

### Functions as Return Values

What if we need to return a new function, a common technique in functional languages so what about the canonical example
of creating a new function that adds a fixed number to it's future argument. It should look something like this:

```rust
fn add_n(n: i32) -> impl Fn(i32) -> i32 {
    fn add(v: i32) -> i32 {
        v + n
    }
    add
}
```

except that we get the following error from the compiler.

```text
error[E0434]: can't capture dynamic environment in a fn item
   --> src/fn_traits.rs:129:13
    |
129 |         v + n
    |             ^
    |
    = help: use the `|| { ... }` closure form instead
```

We are breaking the lexical scoping of Rust, `n` is not available for use in the inner function so a _dynamic 
environment_ (the set of bindings and values available to the function `add`) cannot be constructed without `n`. So we 
use a closure which is a way of explicitly capturing a dynamic environment and returning a value that implements `Fn`.
In this specific case, because n goes out of scope at the end of `add_n` we use the keyword `move` to ensure the value
is moved into the environment rather than borrowed.

```rust
fn add_n(n: i32) -> impl Fn(i32) -> i32 {
    move |v| v + n
}

let add_3 = add_n(3);
println!("{}", add_3(4));
```

So, in writing the signature for `add_n` we used the `impl Trait` form as in `wrapper_2b` above. Does this mean that all 
of the same type forms can be used to define the type returned from `add_n`? It turns out no, it's more complicated but 
if we try to create functions to return callbacks for our wrapper function above we see all sorts of compiler complaints.


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

So, the upshot of all of this, if you are describing the type of an argument to a function use either `impl Trait` or 
trait bounds and if you are describing the return type use `impl Trait`. If you want to create a new `type` for your
functions, well just don't.

Here are some pretty good words on `impl Trait` from [The Edition Guide](https://doc.rust-lang.org/nightly/edition-guide/rust-2018).

> With impl Trait, you're saying "hey, some type exists that implements this trait, but I'm not gonna tell you what it
> is." So now, the caller can't choose, and the function itself gets to choose.


> As previously mentioned, as a start, you will only be able to use impl Trait as the argument or return type of a free 
> or inherent function. However, impl Trait can't be used inside implementations of traits, nor can it be used as the 
> type of a let binding or inside a type alias. Some of these restrictions will eventually be lifted. 

## Documentation Links

* [Trait System](https://doc.rust-lang.org/nightly/edition-guide/rust-2018/trait-system/index.html) in The Edition Guide.

* [`std::boxed::Box`](https://doc.rust-lang.org/std/boxed/struct.Box.html); A pointer type for heap allocation.
* [`std::marker::Sized`](https://doc.rust-lang.org/std/marker/trait.Sized.html); Types with a constant size known at 
  compile time.
* Function Pointers & Traits
  * Primitive type [`fn`](https://doc.rust-lang.org/std/primitive.fn.html); Function pointers, like `fn(usize) -> bool`.
  * [`std::ops::Fn`](https://doc.rust-lang.org/std/ops/trait.Fn.html); The version of the call operator that takes an 
    immutable receiver. Instances of `Fn` can be called repeatedly without mutating state.
  * [`std::ops::FnMut`](https://doc.rust-lang.org/std/ops/trait.FnMut.html); The version of the call operator that takes 
    a mutable receiver. Instances of `FnMut` can be called repeatedly and may mutate state.
  * [`std::ops::FnOnce`](https://doc.rust-lang.org/std/ops/trait.FnOnce.html); The version of the call operator that 
    takes a by-value receiver. Instances of `FnOnce` can be called, but might not be callable multiple times. Because of 
    this, if the only thing known about a type is that it implements `FnOnce`, it can only be called once.