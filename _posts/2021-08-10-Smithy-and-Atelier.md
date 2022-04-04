---
title: Smithy and Atelier
layout: postx
category: work
tags: [aws, idl, rust, smithy]
---
One of my _side-projects_ is named Atelier and is a Rust implementation of the
AWS Smithy IDL framework. [AWS Smithy](https://github.com/awslabs/smithy) is a
framework around a language and runtime neutral
[IDL](https://en.wikipedia.org/wiki/Interface_description_language) for
service definition. It consists of a semantic model, a native IDL
representation, and a mapping to JSON.

The goal of [Atelier](https://github.com/johnstonskj/rust-atelier) is to
replicate these core components of Smithy in Rust but also to experiment in
alternate integrations such as the representation of the semantic model in
RDF. This could provide the basis for Rust code generation from a Smithy model
and other tooling opportunities. This has been a major effort for myself as
well as another ex-Amazon PE with full support from the awesome Smithy team. 

So far Atelier has been used internally for some experimental tooling;
hopefully, something we can also open up on Github some day.

