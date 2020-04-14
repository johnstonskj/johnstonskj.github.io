---
title: Error Handling (Rustioms)
layout: postx
category: code
tags: [rust, errors, idioms]
---

Error detection, handling, and recovery is a really hard problem, and while most of the time it's a face-palm moment,
sometimes they can be quite [spectacular](https://www.bugsnag.com/blog/bug-day-ariane-5-disaster). There are a lot of
different techniques from low-level to system-wide that can be applied but we're going to look at the idioms and tools 
for error propagation in Rust (so we won't cover [Crash-Only](https://en.wikipedia.org/wiki/Crash-only_software) 
approaches).

A common C language approach, which was either influenced by or influenced OS API design is the error signal and 
separate error identification. So, for POSIX it's often a magic value that indicates a function has failed but to know 
why you have to find out from somewhere else. From the man page for the Linux `open` function:

```text
RETURN VALUE
       open(), openat(), and creat() return the new file descriptor, or -1
       if an error occurred (in which case, errno is set appropriately).
```

Which, it turns out, is pretty much the same for the Windows API:

> **Return value**
>
> If the function succeeds, the return value specifies a file handle to use when performing file I/O. To close the file, 
> call the CloseHandle function using this handle.
>
> If the function fails, the return value is **HFILE_ERROR**. To get extended error information, call GetLastError.

On the other hand exceptions are another approach to separate failures from normal execution. For example  
[Taligent](https://root.cern.ch/TaligentDocs/TaligentOnline/DocumentRoot/1.0/Home/index.html) (defunct) was a pure C++ 
API and relied on exceptions for all operations so either succeed or crash-out.

> **Member Function: TFileStream::TFileStream**
>
> **Return Value:**
>
> None.
>
> **Exceptions:**
>
> Throws TFileSystemEntityNotAvailable if the entity cannot be found. Throws TFileSystemAccessDenied if the caller is 
> not privileged to open the file with the requested permissions.

This is similar in nature to the Java `File`, `FileInputStream`, and `FileOutputStream` API. Java is also a language that makes
extensive use of the [`null` reference](https://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions) as a way to 
signal missing values, erroneous execution, and errors. It's not clear that the use of `null` hasn't introduced more
errors over the years than it's use has saved us from.

Both [Go](https://golang.org/) and [Python](https://www.python.org/) use 2-tuples for return values, with one part the 
success/error flag and the other the returned value (or nothing). Go goes further and this is a common idiom throughout
whereas it is an uncommon but useful feature in some Python libraries.

## Option and Result

Rust has two related types used throughout the standard library and 3rd party crates. These two types are similar in
many ways and they are used to support three separate classes of error conditions in Rust.

1. Those conditions where there may, or many not be a response value. For example `HashMap::get()` returns either a
   value for the provided key or it returns a value that indicates that the key was not present in the map. This uses
   the `Option` enumeration, with `Some(v)` denoting a present value and `None` denoting, well there was no value.
1. Those conditions where an error occurs and the response may either be a success value or some value representing
   the error. This uses the `Result` enumeration, with `Ok(v)` denoting a success value and `Err(e)` denoting the error.
1. There are conditions however where either there is no way to return an `Option` or a `Result`, or where there is no way
   to handle the condition gracefully and Rust provides the `panic!()` macro. For example, using the index syntax with
   a hashmap, `hash_map[key]` the response is either the associated value or the function panics.

`Result` is not only an elegant way to bring a common approach to the language but it is also really effective.

```rust
// make my own result type in a library (see below for error types)
type Result<T> = std::resut::Result<T, MyError>;

// use `map` to manipulate an `Ok` value
foo.map(|n| n - 1)

// or `map_err` to manipulate an `Err` value (see below for error types)
foo.map_err(|e| MyError::Wrapper(e))

// use `?` call another function and return on error
let foo = do_this(do_that(v)?)?;

// Map an Option<&T> to an Option<T>
let foo = do_this.cloned();
```

The `?` suffix on a function call is some syntactic sugar, it says that the function returns a `Result`, if it `is_ok()`
 then return the value, if not pass the error immediately out of the enclosing function.

`Option` is very similar, check out the functions on both, there's way more than the `is_some()` and `is_none()` we all 
pick up early on. Also the ability to turn an `Option` into a `Result` with `ok_or()` / `ok_or_else()`, or the `Result` 
functions `ok()` and `err()` which return `Option`s show the interaction between these. There is even a `transpose()` 
function on both types that transform between them.

## Error Types

Basically defining your own errors is pretty simple, start with a plain enum and a variant for each error case. As with 
any other enum you can add values to the variant which helps with debugging. Here we call the type `MyError`, this is 
for clarity here, you would normally just call it `Error`. Note that this is the pedantic version of this, it seems like 
a lot of work, but a lot of it is reusable.

```rust
#[derive(Debug)]
pub enum MyError {
    NotFound,
    DidntLikeThis(String),
    IOError(std::io::Error),
}
```

Note, usually you'd also derive `Clone` and `PartialEq` for an enum, but the `std::io::Error` type doesn't support 
those so we can't.

To get a description of the error, we simply use the common `Display` trait.

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

One of the cool features of Rust is the ability to do explicit type conversion (not really coercion as it is always 
intentional). This is done with the `From`, `Into`, `TryFrom`, `TryInto` traits. For our case we'd like to easily be 
able to convert from IO errors to my error type.

```rust
impl From<std::io::Error> for MyError {
    fn from(err: Error) -> Self {
        MyError::IOError(err)
    }
}
```

This allows us to write code like the following, the inner `read_to_string` returns a different error, but when we 
unwrap the result using '?' the compiler realizes there is a `From` implementation that can bridge the two types and 
applies it for us.

```rust
fn read_into_str(file_name: &str) -> MyResult<String> {
    use std::fs;
    Ok(fs::read_to_string(file_name)?)
}
```

Finally, we implement the standard library `Error` trait. Unless you wrap underlying errors (as we do with the IO error), 
this is usually an empty implementation, but for our example we must implement `source`. This allows a standard way to 
walk down errors as they are abstracted by subsequent layers.

```rust
impl std::error::Error for MyError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        match self {
            MyError::IOError(err) => Some(err),
            _ => None,
        }
    }
}
```

Having walked through the roll-your-own `Error` process, there are some great crates such as 
[failure](https://crates.io/crates/failure) that can do all of this and more for you. 

Typically I have a top-level `error` module that only contains the above set of definitions and implementations, it 
also implements the unsafe `Send` and/or `Sync` traits if the compiler doesn't do it automatically to allow error values 
to be handed across threads. You also add your own `Result` type as well, as in the example in the first section.

## Documentation Links

* [`std::clone::Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html); A common trait for the ability to explicitly duplicate an object.
* [`std::cmp::PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html); Trait for equality comparisons which are partial equivalence relations.
* [`std::convert::From`](https://doc.rust-lang.org/std/convert/trait.From.html); Used to do value-to-value conversions while consuming the input value. It is the reciprocal of `Into`.
* [`std::error::Error`](https://doc.rust-lang.org/std/error/trait.Error.html); Error is a trait representing the basic expectations for error values, i.e., values of type `E` in `Result<T, E>`.
* [`std::fmt::Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html); Format trait for an empty format, `{}`.
* [`std::io::Error`](https://doc.rust-lang.org/std/io/struct.Error.html); The error type for I/O operations of the `Read`, `Write`, `Seek`, and associated traits.
* [`std::option::Option`](https://doc.rust-lang.org/std/option/enum.Option.html); The Option type.
* [`std::result::Result`](https://doc.rust-lang.org/std/result/enum.Result.html); Result is a type that represents either success (`Ok`) or failure (`Err`).
