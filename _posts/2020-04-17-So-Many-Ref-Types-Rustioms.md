---
title: So Many Ref Types (Rustioms)
layout: postx
category: code
tags: [rust, ref, idioms]
---

Rust 101 always includes a discussion of `T` vs. `&T` where the first is an owner value and the latter an immutable 
reference to a value, then there's `&mut T` which is a mutable reference to that value (and we can only have one of these). But 
this isn't the whole story. 

In [_It's Traits (Almost) All the Way Down_]({% post_url 2020-04-10-Its-Traits-All-The-Way-Down-Rustioms %}), we met 
`Box<T>` which takes a value and moves it to the heap. In that case it was required to allow us to pass back a reference
type, but another example often comes up, recursive data structures. If a struct contains a field of the same type then
it implies that any value also contains a value of the same time _ad infinitum_, and Rust doesn't like infinities. So, 
we add `Box`, because it's value is a fixed size (a pointer to it's enclosed heap value) and so we avoid infinity.

So clearly the following _does not_ work.
```rust
pub struct IllegalRecursive {
    // other fields...
    next_one: IllegalRecursive,
}
```

And the compiler even gives us a hint as to the next step.

```text
1 | pub struct IllegalRecursive {
  | ^^^^^^^^^^^^^^^^^^^^^^^^^^^ recursive type has infinite size
2 |     // other fields...
3 |     next_one: IllegalRecursive,
  |     -------------------------- recursive without indirection
  |
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to make `structs::IllegalRecursive` representable
```

We could use a reference, `&RefRecursive` but as Rust 101 has certainly made clear we will have to deal with lifetimes
and so the next version of our struct looks like this.

```rust
pub struct RefRecursive<'a> {
    // other fields...
    next_one: &'a RefRecursive<'a>,
}
```

This is a perfectly legal approach and for some cases it will be the appropriate technique but `Box` is a far simpler 
looking and easier to manage approach.

```rust
pub struct BoxRecursive {
    // other fields...
    next_one: Box<BoxRecursive>,
}
```

The compiler also mentioned `Rc`, another type for holding a reference to a value, at first look in fact `Box<T>` and 
`Rc<T>` look very similar. The differences are interesting though, and `Rc` has a trick that `Box` does not, the ability 
to track ownership across multiple `Rc` copies of the same enclosed value. So, when you use `borrow()` or `get_mut()` it
is ensuring the borrow checker rules apply regardless of how many wrapper values exist. 

## The Rc + RefCell Pattern

One interesting use of `Rc` is to pair it with another wrapper/reference type called `RefCell`. This wrapper is 
interesting in itself as it provides something termed _interior mutability_, basically you can mutate a value behind a
ref cell even if the ref cell itself isn't necessarily mutable. This is an extension of an idea presented in the 
documentation for [`RefCell::borrow_mut()`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.borrow_mut);
in this case we have a struct that wraps it's inner value in a `Rc` around a `RefCell` around some actual value.

If we were to look at the signature `fn increment(thing: &Thing)` we wouldn't imagine it's possible that we could mutate
`thing` as it isn't a mutable reference. The key here is that when we call `borrow_mut` inside increment it is performing
a check of all known references to the same value and determining the borrow checker rules dynamically and as there are 
no existing references it allows mutation. 

```rust
#[derive(Clone, Debug)]
pub struct Thing {
    inner: Rc<RefCell<u64>>,
}

pub fn increment(thing: &Thing) {
    let inner = &thing.inner;
    let value: u64 = inner.borrow().to_owned();
    *inner.borrow_mut() = value + 1;
}

#[test]
fn test_add() {
    let initial = Thing {
        inner: Rc::new(RefCell::new(10)),
    };
    println!("{:?}", initial);
    let another = initial.clone();
    increment(&another);
    println!("{:?}", initial);
}
```

Another key to this magic is that to create a new reference we simply use `clone()` as we might expect, so in this 
example we increment `another` which is a clone, and we see that the inner value is mutated via the `initial` reference.

## Cows are useful

I had seen the `Cow<T>` type in a few APIs and dug into the documentation for this _clone-on-write_ reference type, but 
also found this fantastic article [`The Secret Life of Cows`](https://deterministic.space/secret-life-of-cows.html) by 
Pascal Hertleif. I'll not spend more time on cows here, they're a related topic but have some really nice unique 
capabilities.

## Documentation Links

Definitely go read §15, [Smart Pointers](https://doc.rust-lang.org/stable/book/ch15-00-smart-pointers.html) in The Rust 
Programming Language.

* [`std::boxed::Box`](https://doc.rust-lang.org/std/boxed/struct.Box.html); `Box<T>`, casually referred to as a 'box', 
  provides the simplest form of heap allocation in Rust. Boxes provide ownership for this allocation, and drop their 
  contents when they go out of scope.
* [`std::rc::Rc`](https://doc.rust-lang.org/std/rc/struct.Rc.html) and 
  [`std::rc::Weak`](https://doc.rust-lang.org/std/rc/struct.Weak.html); The type `Rc<T>` provides shared ownership of a 
  value of type `T`, allocated in the heap. Invoking clone on `Rc`  produces a new pointer to the same allocation in the
  heap. When the last `Rc` pointer to a given allocation is  destroyed, the value stored in that allocation (often 
  referred to as "inner value") is also dropped. A cycle between `Rc` pointers will never be deallocated. For this 
  reason, `Weak` is used to break cycles. For example, a tree could have strong `Rc` pointers from parent nodes to 
  children, and `Weak` pointers from children back to their parents.
* [`std::sync::Arc`](https://doc.rust-lang.org/std/sync/struct.Arc.html) and 
  [`std::sync::Weak`](https://doc.rust-lang.org/std/sync/struct.Weak.html); A thread-safe reference-counting pointer. 
  'Arc' stands for 'Atomically Reference Counted'. Shared references in Rust disallow mutation by default, and `Arc` is 
  no exception: you cannot generally obtain a mutable reference to something inside an `Arc`. If you need to mutate 
  through an `Arc`, use `Mutex`, `RwLock`, or one of the Atomic types.
* [`std::cell::Cell`](https://doc.rust-lang.org/std/cell/struct.Cell.html) and 
  [`std::cell::RefCell`](https://doc.rust-lang.org/std/cell/struct.RefCell.html); Values of the `Cell<T>` and 
  `RefCell<T>` types may be mutated through shared references (i.e. the common `&T` type), whereas most Rust types can 
  only be mutated through unique (`&mut T`) references. We say that `Cell<T>` and `RefCell<T>` provide 'interior 
  mutability', in contrast with typical Rust types that exhibit 'inherited mutability'.
* [`std::cell::UnsafeCell`](https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html); `UnsafeCell<T>` is a type that 
  wraps some `T` and indicates unsafe interior operations on the wrapped type. Types with an `UnsafeCell<T>` field are 
  considered to have an 'unsafe interior'. The `UnsafeCell<T>` type is the only legal way to obtain aliasable data that 
  is considered mutable.
* [`std::borrow::Cow`](https://doc.rust-lang.org/std/borrow/enum.Cow.html); A clone-on-write smart pointer. The type 
  `Cow` is a smart pointer providing clone-on-write functionality: it can enclose and provide immutable access to 
  borrowed data, and clone the data lazily when mutation or ownership is required. The type is designed to work with 
  general borrowed data via the `Borrow` trait.
