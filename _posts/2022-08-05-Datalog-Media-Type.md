---
title: Datalog Media Type
layout: postx
category: code
tags: [datalog, iana, rust]
---

I have been working on a [Datalog](https://en.wikipedia.org/wiki/Datalog) Rust
project for a while now, which I hope to talk about another time, however in
writing a parser it was clear there is no clear standard representation. There
are a number of ways to describe a particular rule and there are various
extensions to the core Datalog language that introduce new syntax and syntax
variants. 

So, being somewhat of a masochist, I set out to write a document that
describes a standard representation as well as mechanisms for language
extensions to be added. This slowly morphed into a pretty detailed and
standards-like document -- [DATALOG-TEXT: Datalog Text
Representation](https://datalog-specs.info/vnd_datalog_text/abstract.html).

As it seemed like a pretty useful thing, I then applied to IANA/IETF for a
registered media type,
[`application/vnd.datalog`](https://www.iana.org/assignments/media-types/application/vnd.datalog)
which was approved on July 20, 2022.
