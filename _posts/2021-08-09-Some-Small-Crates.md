---
title: Some Small Crates
layout: postx
category: code
tags: [rust]
---

While a few big projects have taken a lot of time in the last year or so, a number of small supporting crates have been required. Hopefully these may be of use to others, certainly a few have been reused in multiple ways in some of those larger projects.

## Dynamic Generic PlugIns

The [dygpi](https://crates.io/crates/dygpi) crate implements a simple plugin model that allows for loading of implementations from external dynamic libraries at runtime.

1. The plugin _host_ defines a concrete type, the plugin _type_.
   1. The plugin _type_ MUST implement the trait `Plugin`.
   1. It MAY be preferable to define the plugin _type_ in a separate plugin API crate that both the _host_ and _provider_ depend upon.
1. The plugin _provider_ (or _library_) crate MUST set `crate-type` to "dylib" and "rlib" in their cargo configuration.
1. The plugin _provider_ MUST implement a function, named `register_plugins`, which is passed a registrar object to register any instances of the plugin _type_.
   1. A plugin _provider_ MAY use an alternate name for the registration function, but this must be provided to the plugin manager via the `set_registration_fn_name` method.
1. The plugin _host_ then uses the `PluginManager` to load libraries, and register plugins, that have the same type as the plugin _type_.
1. The plugin _host_ MAY then use plugin manager’s `get` method to fetch a specific plugin by _id_, OR use plugin manager’s `plugins` method to iterate over all plugins.

Overriding the plugin registration function allows a plugin _host_ to provide plugins of different types by using separate registration functions for each type.

## String-base new types

The [newstr](https://crates.io/crates/newstr) crate provides simple macros that generate `String` based new types. The two primary macros implement the validity rules for the new type by either 1) providing a predicate that is used by the `is_valid` associated function, or 2) providing a function to parse and return a string which is then called by `FromStr::from_str`.

Both of these methods produce a new type, with the following:

1. An associated predicate function `is_valid` that returns `true` if the string provided would be a valid value for the type.
1. This type derives implementations of `Clone`, `Debug`, `PartialEq`, `PartialOrd`, `Eq`, `Ord`, and `Hash`.
1. An implementation of `Display` for `T` that simply returns the inner value.
1. An implementation of `From<T>` for `String`.
1. An implementation of `Deref` for `T` with the target type `str`.
1. An implementation of `FromStr`.

Additional user-required traits can also be added to the macro to be derived by the implementation.

## Search path file finder

The [search_path](https://crates.io/crates/search_path) crate provides a single `SearchPath` type. This type allows for the finding of files using a series of search directories. 

This is akin to the mechanism a shell uses to find executables using the `PATH` environment variable. It can be constructed with a search path from an environment variable, from a string, or from a list of either string or `Path`/`PathBuf` values. Typically the `find*` methods return the first matching file or directory, but the `find_all` method specifically collects and returns a list of all matching paths.

## Text tree output

The [text_trees](https://crates.io/crates/text_trees) crate is another that will output a tree structure in text. 

Similar to the existing [ascii_tree](https://crates.io/crates/ascii_tree) crate, however it is more flexible in its formatting options.

## Simple document model

The [somedoc](https://crates.io/crates/somedoc) crate provides an abstract model for formatted documents and a trait for serializing.

The `somedoc::model` module provides the model to construct formatted documents.

The `somedoc::write` module contains a number of serializers that generate specific markup formats for different platforms. So far, this includes HTML, LaTeX, and Markdown of different flavors.

A JSON representation of the library's `Document` structure is also provided and can be read as well as written to allow for tool interchange.

## Unique ID generators

The [unique_id](https://crates.io/crates/unique_id) crate provides four simple traits, starting with Generator. This will return successive unique identifiers, the only requirement of an identifier being that it implements PartialEq.

Each implemented generator is in its own feature, by default all of which are included.

* Generator ``RandomGenerator`, in feature random, provides a random number scheme returning u128 values. Depends on the [uuid](https://crates.io/crates/uuid) crate.
* Generator SequenceGenerator, in feature sequence, provides monotonically increasing u64 values in a thread safe manner. Depends on the [atomic_refcell](https://crates.io/crates/atomic_refcell) and [lazy_static](https://crates.io/crates/lazy_static) crates.
* Generator ``StringGenerator`, in feature string, provides random string values. Depends on the [blob-uuid](https://crates.io/crates/blob-uuid) crate.

