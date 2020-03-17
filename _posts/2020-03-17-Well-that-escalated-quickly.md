---
title: Well that escalated quickly
layout: postx
category: code
tags: [rust, xml, dom]
---

I was looking for another side project to keep myself busy when I'm not doing the day job, and specifically I am 
enjoying Rust but still don't feel like I fully understand the borrow checker any why sometimes I code myself into a
corner I can't borrow my way out of. So, find something that forces you to deal with it, a structure with inherent
references, something useful but complex enough to be fun. I was working on something else, which included an XML writer
and which one day was going to need an XML reader also. So I wondered, what about an implementation of the DOM, I know
it, have used it so many times from a bunch of languages, it is a tree structure but has certain links that would make
references a pain, so why not?

Well, I was about to find out some really good answers to _why not_ in short order.

# Read The F-ing Standards (RTFS)

I have read a lot of industry standards in my time, I've even written and contributed to some, I have written software 
that implements standards too. But nothing quite prepared me for the complexity of the task ahead of me.

Obviously one starts with a quick search for "DOM API" and I found [_Document Object Model (DOM) Level 1 
Specification_](https://www.w3.org/TR/REC-DOM-Level-1/) which had the look of a good starting place. Except that, what
you really want is [_Document Object Model (DOM) Level 2 Core Specification_](https://www.w3.org/TR/DOM-Level-2-Core/). It
turns out that the writers of these things realized that they were dealing with (at least) two communities, those using
the DOM as a helpful formalism over a horribly ill-defined and frequently ill-written format namely HTML. But, then
there are those needing an API that is a faithful representation of a language more formally defined and in most cases
written in a valid and reasonable manner namely XML. So, loosely speaking Level 1 (not version 1) is the set of 
interfaces required for HTML and Level 2 is a superset of these interfaces that covers XML. OK, So _Level 2 Core_ it is.

But it doesn't end there. To understand what is a valid name, what constitutes valid content, or implement
_Attribute-Value Normalization_, you really have to dive into 
[_Extensible Markup Language (XML) 1.1 (Second Edition)_](https://www.w3.org/TR/xml11/) (which supersedes
[Extensible Markup Language (XML) 1.0 (Fifth Edition)](https://www.w3.org/TR/REC-xml/) but which you really should keep
handy to understand the differences). To understand the error conditions around `NAMESPACE_ERR` you really need to go 
read [_Namespaces in XML 1.1 (Second Edition)_](https://www.w3.org/TR/xml-names11/) as well as 
[_The "xml" Namespace_](https://www.w3.org/XML/1998/namespace). To understand how to implement the DOM Document's 
`get_element_by_id` method you'll need [_xml:id Version 1.0_](https://www.w3.org/TR/xml-id). There are also 
additional specifications dealing with encoding names, language identifiers, and URIs used for namespaces.
 
Now if you decide to go really nuts and have your processor be schema aware, then you can start on the XML Schema 
family of specifications; 
[_Part 0: Primer Second Edition_](https://www.w3.org/TR/xmlschema-0/),
[_Part 1: Structures Second Edition_](https://www.w3.org/TR/xmlschema-1/),
[_Part 2: Datatypes Second Edition_](https://www.w3.org/TR/xmlschema-2/),
[_W3C XML Schema Definition Language (XSD): Component Designators_](https://www.w3.org/TR/xmlschema-ref/), and
[_Processing XML 1.1 documents with XML Schema 1.0 processors_](https://www.w3.org/TR/xml11schema10/).

My head hurts already.

# A few ways that won't work

> _“I have not failed. I've just found 10,000 ways that won't work.”_ -- Thomas Edison.

Getting to the right implementation model/approach took a few tries. My first was to develop traits that mapped 1:1 
with the specification and a set of structs that could model nodes. This seemed reasonable but I ended up with myself
wrapped up in lifetimes, trying to copy references around and basically making my own life miserable. The second 
approach was to have a struct for each interface, that also ended up in a mess trying to keep track of lifetimes as
I had all kinds of `&'a Node` fields in these structs. 

After a while I decided that honestly doing this from first principals wasn't productive so what had other people done? 
So, digging around I found the [`amxml`](https://crates.io/crates/amxml) crate and inside it's `dom` module a neat trick:

```rust
pub struct NodePtr {
    rc_node: RcNode,
}

type RcNode = Rc<Node>;

fn wrap_rc_clone(rc_node: &RcNode) -> NodePtr {
    return NodePtr{ rc_node: Rc::clone(rc_node) };
}

// ...

struct Node {
    node_type: NodeType,
    order: Cell<i64>,
    name: String,
    value: String,
    parent: Option<RefCell<Weak<Node>>>,
    children: RefCell<Vec<RcNode>>,
    attributes: RefCell<Vec<RcNode>>,
}
```

The `NodePtr` values can easily be created, and as their only member is an `Rc<Node>` ([`Rc`](https://doc.rust-lang.org/std/rc/struct.Rc.html)) 
there can be a number of these structs  sharing the same `Node` value. The vector of `children` in each `Node` allow us 
to retain ownership with `Rc`, but this doesn't work well for `parent` (or for `owning_document` which isn't present in 
`amxml`). However, a `Weak<Node>` ([`Weak`](https://doc.rust-lang.org/std/rc/struct.Weak.html)) created by `Rc::downgrade` 
works well to capture these additional references. Additionally, [`RefCell`](https://doc.rust-lang.org/std/cell/struct.RefCell.html) 
is added to allow mutation of elements in subtle ways within the implementation.

The corresponding types in xml_dom are split between a generic `rc_cell` module:

```rust
pub struct RcRefCell<T: Sized> {
    inner: Rc<RefCell<T>>,
}

pub struct WeakRefCell<T: Sized> {
    inner: Weak<RefCell<T>>,
}
```

and the `node_impl` module (note that the `WeakRefNode` is not exposed to clients):

```rust
pub type RefNode = RcRefCell<NodeImpl>;

pub(crate) type WeakRefNode = WeakRefCell<NodeImpl>;
```

Taking an approach where there is a single structure for all node types, `NodeImpl`, the question is then how to 
store all the node-type specific details. Here is the final form of `NodeImpl`.

```rust
pub(crate) enum Extension {
    None,
    Document {
        i_xml_declaration: Option<XmlDecl>,
        i_document_element: Option<RefNode>,
        i_document_type: Option<RefNode>,
        i_id_map: HashMap<String, WeakRefNode>,
        i_options: ProcessingOptions,
    },
    DocumentType {
        i_entities: HashMap<Name, RefNode>,
        i_notations: HashMap<Name, RefNode>,
        i_public_id: Option<String>,
        i_system_id: Option<String>,
        i_internal_subset: Option<String>,
    },
    Element {
        i_attributes: HashMap<Name, RefNode>,
        i_namespaces: HashMap<Option<String>, String>,
    },
    Entity {
        i_public_id: Option<String>,
        i_system_id: Option<String>,
        i_notation_name: Option<String>,
    },
    Notation {
        i_public_id: Option<String>,
        i_system_id: Option<String>,
    },
}

pub struct NodeImpl {
    pub(crate) i_node_type: NodeType,
    pub(crate) i_name: Name,
    pub(crate) i_value: Option<String>,
    pub(crate) i_parent_node: Option<WeakRefNode>,
    pub(crate) i_owner_document: Option<WeakRefNode>,
    pub(crate) i_child_nodes: Vec<RefNode>,
    pub(crate) i_extension: Extension,
}
```

# It's Alive!!

I have by now checked in a couple of versions and created the crate [xml_dom](https://crates.io/crates/xml_dom), and 
while not all things work as they should, some very basic things do.

```rust
use xml_dom::*;
use xml_dom::convert::*;

let implementation = get_implementation();
let mut document_node = implementation
    .create_document("http://www.w3.org/1999/xhtml", "html", None)
    .unwrap();
println!("document 1: {:#?}", document_node);

let document = as_document_mut(&mut document_node).unwrap();
let mut root_node = document.document_element().unwrap();

let root = as_element_mut(&mut root_node).unwrap();
root.set_attribute("lang", "en");
let _head = root.append_child(document.create_element("head").unwrap());
let _body = root.append_child(document.create_element("body").unwrap());

let xml = document_node.to_string();
println!("document 2: {}", xml);
```

Now, it's not necessarily pretty, but in general DOM programming rarely is. There's a lot of casting between node types
and so on, but it works. Also note the need for a non-specification `get_implementation` function to deal with the 
following in §1.1.2. Memory Management:

> The DOM Level 2 API does not define a standard way to create `DOMImplementation` objects; DOM implementations must 
> provide some proprietary way of bootstrapping these DOM interfaces, and then all other objects can be built from there.

# That got big, fast 

After checking in code to add support for XML declarations (why isn't that a part of the DOM?) and was considering 
writing this post, I actually wondered, how big had this gotten? Well, it turns out that [scc](https://github.com/boyter/scc)
seems to think I wrote ~7 months of work in the last two and a half weeks. 

```text
───────────────────────────────────────────────────────────────────────────────
Language                 Files     Lines   Blanks  Comments     Code Complexity
───────────────────────────────────────────────────────────────────────────────
Rust                        26      8029      737      2695     4597        532
License                      1        21        4         0       17          0
Markdown                     1        84       21         0       63          0
TOML                         1        14        1         0       13          0
gitignore                    1         4        1         0        3          0
───────────────────────────────────────────────────────────────────────────────
Total                       30      8152      764      2695     4693        532
───────────────────────────────────────────────────────────────────────────────
Estimated Cost to Develop $136,967
Estimated Schedule Effort 7.207851 months
Estimated People Required 2.250957
───────────────────────────────────────────────────────────────────────────────
```

What it doesn't know is that those 2,695 lines of comments are _mostly_ copied from the various W3C specifications 
either into the documentation produced by `cargo doc` or internally as reminders about various rules and validation 
requirements. But still, I didn't know I typed that fast.

Also, where can I apply for the $137K?
 
# And yet, so much more

The DOM doesn't actually cover the XML declaration (_Definition: XML 1.1 documents MUST begin with an **XML 
declaration** which specifies the version of XML being used._) which seems awkward if it's a required part of XML 1.1.

Now, how do we parse text into the DOM? I'd rather not go further down this rabbit hole and build my own parser, there 
are some existing crates out there, but not all of them parse all the components required to construct a high-fidelity
DOM.

Is the implementation of `Display` for `RefNode` enough to write XML out again?

...

