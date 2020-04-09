---
title: It's Traits (Almost) All the Way Down (Rustioms)
layout: post
categories: [code]
tags: [rust, traits, idioms]
---

Traits are seen in a lot of places in the standard library, some are markers (the presence or not of an implementation
determines some other behavior), capabilities that the compiler may use for type conversion for example. The fact that
traits are a very light-weight mechanism _and_ support default method implementation means the implementation of sets of
traits are inexpensive to use but add significant value. Library writers definitely should to look at existing types in
the standard library and see what traits they implement.

For example, here are some ways in which we use common traits as library writers.

1. It's recommended practice to always implement `std::fmt::Debug` for all public types, this is simple and usually 
   done using the `#[derive(Debug)]` attribute on your type.
1. `std::clone::Clone` and `std::marker::Copy` are used by the compiler to determine how values may be passed between 
   functions.
1. Traits are used to provide operator overloading, rather than most languages where you have to create a method called 
   "+" somehow, in Rust you just implement the `std::ops::Add` trait for your type. All operator traits are found in 
    `std::ops`.
1. Relational operators are also implemented using traits such as `PartialEq` and `Eq` for "=" and "!=", `PartialOrd` 
   and `Ord` for "<" and ">". These can be found in `std::cmp`.
1. We mentioned `From`, `Into`, `TryFrom` and `TryInto` in `std::convert` as ways to convert between types, there's 
   also `std::str::FromStr` and `std::string::ToString`:
   1. `FromStr` is useful to implement `MyType::from_str()`, but the presence of this type also allows you to use 
      `str::parse()` to achieve the same result.
   1. `ToString` is rarely necessary if you implement `Display`, which you usually should, so that `to_string()` is 
      available for any type that implements Display.
1. Consider implementing `std::default::Default` for your types if there is a valid default value, some functions are 
   bound to types that can be created by default.
1. If you are making some kind of container type, consider implementing `std::borrow::Borrow` or 
   `std::borrow::BorrowMut` to access contained values.
   
## Trait Bounds

Let's assume we want to create some new container type, we will also assume you have some good reason for this, rather 
than using one of the existing ones. Now, the trade-off with generics, is to try and make your type as flexible as 
possible, but to provide as much expected/common behavior as you can. This is where trait bounds help, you can define 
the type itself to know nothing about it's content, but implement core traits with bounds so that they apply if the 
compiler knows the contained type is able to support it.

Taking the first impl in the  example below, `impl<T: Clone> Clone for MyContainer<T>` can be read as "implement `Clone` 
for `MyContainer`, IFF the type `T` implements `Clone`". So we can now see that we have implemented a bunch of core 
traits, `Clone`, `Copy`, `Debug`, `Display`, `From`, `Borrow`, and `BorrowMut` for our type, all dependent on `T` 
supporting the same. 

We also have two `impl` blocks for `MyContainer` itself, one is unconstrained and provides `map()` and `take()`, the 
other provides a `contains()` function, but only if `T` supports "==" by implementing `PartialEq`.

```rust
pub struct MyContainer<T> {
    inner: T,
}

impl<T: Clone> Clone for MyContainer<T> {
    fn clone(&self) -> Self {
        Self {
            inner: self.inner.clone(),
        }
    }
}

impl<T: Copy> Copy for MyContainer<T> {}

impl<T: Debug> Debug for MyContainer<T> {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        if f.alternate() {
            write!(f, "MyContainer({:#?})", self.inner)
        } else {
            write!(f, "MyContainer({:?})", self.inner)
        }
    }
}

impl<T: Display> Display for MyContainer<T> {
    fn fmt(&self, f: &mut Formatter<'_>) -> std::fmt::Result {
        if f.alternate() {
            write!(f, "MyContainer({:#})", self.inner)
        } else {
            write!(f, "MyContainer({:})", self.inner)
        }
    }
}

impl<T: Default> Default for MyContainer<T> {
    fn default() -> Self {
        Self {
            inner: Default::default(),
        }
    }
}

impl<T> From<T> for MyContainer<T> {
    fn from(inner: T) -> Self {
        Self { inner }
    }
}

impl<T> Borrow<T> for MyContainer<T> {
    fn borrow(&self) -> &T {
        &self.inner
    }
}

impl<T> BorrowMut<T> for MyContainer<T> {
    fn borrow_mut(&mut self) -> &mut T {
        &self.inner
    }
}

impl<T: PartialEq> MyContainer<T> {
    pub fn contains(&self, test: &Self) -> bool {
        self.inner == test.inner
    }
}

impl<T> MyContainer<T> {
    pub fn map<U, F: FnOnce(T) -> U>(self, f: F) -> MyContainer<U> {
        MyContainer {
            inner: f(self.inner),
        }
    }

    pub fn take(self) -> T {
        self.inner
    }
}
```

Now, all of that may seem like a lot of boiler-plate code, and it was, the following provides `Clone`, `Copy`, `Debug`, 
and `Default` using derive. But, to achieve this you have to constrain `T` at this early stage to also be those things. 
The composition of traits in Rust, while maybe adding some verbosity to library types is a trade-off for the level of 
flexibility you get and the ability to compose-in functionality. 

```rust
#[derive(Clone, Copy, Debug, Default)]
pub struct MyOtherContainer<T: Clone + Copy + Debug + Default> {
    inner: T,
}
```

## Num Traits

One area where the language is lacking, in my opinion, is in not having traits for certain type classes. For example, 
I can have an trait bound to the `Add` trait to ensure I can use "+" on a generic, but I can't have a trait bound to
an integer (regardless of size). There is a way to plug this gap, an excellent crate, 
[num-traits](https://crates.io/crates/num-traits), that I have used but it seems like something pretty core.

## Documentation Links

* Commonly derived
  * [`std::fmt::Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html); Debug should format the output in a programmer-facing, debugging context.
  * [`std::clone::Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html); A common trait for the ability to explicitly duplicate an object.
  * [`std::marker::Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html); Types whose values can be duplicated simply by copying bits.
  * [`std::cmp::PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html); Trait for equality comparisons which are partial equivalence relations.
  * [`std::cmp::Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html); Trait for equality comparisons which are equivalence relations.
  * [`std::cmp::PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html); Trait for values that can be compared for a sort-order.
  * [`std::cmp::Ord`](https://doc.rust-lang.org/std/cmp/trait.Ord.html); Trait for types that form a total order.
* Operations
  * [`std::ops::Add`](https://doc.rust-lang.org/std/ops/trait.Add.html); The addition operator `+`.
  * [`std::ops::Index`](https://doc.rust-lang.org/std/ops/trait.Index.html); Used for indexing operations (`container[index]`) in immutable contexts.
* Conversions
  * [`std::convert::From`](https://doc.rust-lang.org/std/convert/trait.From.html); Used to do value-to-value conversions while consuming the input value. It is the reciprocal of `Into`.
  * [`std::convert::Into`](https://doc.rust-lang.org/std/convert/trait.Into.html); A value-to-value conversion that consumes the input value. The opposite of `From`.
  * [`std::str::FromStr`](https://doc.rust-lang.org/std/str/trait.FromStr.html); Parse a value from a string.
  * [`std::string::ToString`](https://doc.rust-lang.org/std/string/trait.ToString.html); A trait for converting a value to a `String`.
  * [`std::convert::TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html); Simple and safe type conversions that may fail in a controlled way under some circumstances. It is the reciprocal of `TryInto`.
  * [`std::convert::TryInto`](https://doc.rust-lang.org/std/convert/trait.TryInto.html); An attempted conversion that consumes `self`, which may or may not be expensive.
* Container-like
  * [`std::borrow::Borrow`](https://doc.rust-lang.org/std/borrow/trait.Borrow.html); A trait for borrowing data.
  * [`std::borrow::BorrowMut`](https://doc.rust-lang.org/std/borrow/trait.BorrowMut.html); A trait for mutably borrowing data.
* [`std::default::Default`](https://doc.rust-lang.org/std/default/trait.Default.html); A trait for giving a type a useful default value.
* [`std::fmt::Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html); Format trait for an empty format, `{}`.
