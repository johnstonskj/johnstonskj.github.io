---
title: "Quick and Dirty, in Rust?"
layout: postx
category: code
tags: [rust]
---

So, I had a problem yesterday, just needed to execute a few LDAP queries to get the managers of a list of folk's login IDs. Seems pretty simple, and I've done a bunch of LDAP "stuff" before so seemed like a simple enough job. Well, my first choice for simple jobs is to turn to Racket but in this case the LDAP support wasn't what I really wanted. Next choice is a shell (bash or zsh) script using `ldapsearch`, right?

All, I need to do is take a bunch of user IDs, call `ldapsearch` which returns results in a regular "`key: value\n`" with a line for each of the attributes listed on the command line.

```bash
$ ldapsearch -LLL -x -h $LDAP_HOST -p $LDAP_PORT -b $LDAP_BIND \
  "(uid=$uid)" \
  uid description manager jobfamily joblevel 
```

This is easy enough to accomplish in a loop, but (and there's always at least one) I need to combine each result into a single CSV (comma-separated value) file with one line per UID. Now I can start messing with `cut`, `tr`, or maybe `sed` but by now this effort was no longer quick, but was definitely getting dirty. OK, so this wasn't working so what next, Python maybe? It does have great libraries and a reputation for ease of use, and I've done it plenty of times before. Or, which is a wild idea, Rust? The language that often gets bashed for being complicated (borrow checker, etc.) and slow (coooommmmmmpillller) to develop in.

First step as always: 

```bash
$ cargo init --bin --edition 2021 ldapq
     Created binary (application) package
```

Now we have to do a quick check on [crate.io](https://crates.io/), or alternatively use the [crates-io-cli](https://github.com/Byron/crates-io-cli) tool, for an LDAP package. I like the look of `ldap3`, so we add it to our dependencies.

```toml
[dependencies]
ldap3="0.9"
```

I've used the `csv` crate before, and I usually use `structopt` for command-line parsing, so we end up with: 

```toml
[dependencies]
csv = "1.1"
ldap3="0.9"
structopt = "0.3"
```

Step #1, we create a `struct` to model our command-line:

```rust
use structopt::StructOpt;

#[derive(Clone, Debug, StructOpt)]
struct CommandLine {
    #[structopt(short = "c", long)]
    connection: String,

    #[structopt(short = "f", long = "from")]
    query_from: String,

    #[structopt(short = "w", long = "where")]
    query_where: Vec<String>,

    #[structopt(short = "p", long = "project")]
    query_project: Vec<String>,
}
```

Step #2, the `main` function will create an LDAP connection, execute the query and pass the results to the function `format_as_csv`.

```rust
use ldap3::{LdapConn, Scope};
use std::error::Error;

fn main() -> Result<(), Box<dyn Error>> {
    let mut options = CommandLine::from_args();

    let mut connection = LdapConn::new(&options.connection)?;

    let query_where = format!(
        "(|{})",
        options
            .query_where
            .iter()
            .map(|v| format!("({})", v))
            .collect::<Vec<String>>()
            .join("")
    );

    if options.query_project.is_empty() {
        options.query_project = vec![
            "uid".to_string(),
            "description".to_string(),
            "manager".to_string(),
            "jobfamily".to_string(),
            "joblevel".to_string(),
        ];
    }

    let (rs, _res) = connection
        .search(
            &options.query_from,
            Scope::Subtree,
            &query_where,
            options.query_project.clone(),
        )?
        .success()?;

    format_as_csv(options.query_project, rs)?;

    Ok(connection.unbind()?)
}
```

Step #3, read each `ResultEntry`, convert into a `SearchEntry` and then into a CSV row using a `csv::Writer`.

```rust
use ldap3::{ResultEntry, SearchEntry};

fn format_as_csv(headers: Vec<String>, rows: Vec<ResultEntry>) -> Result<(), Box<dyn Error>> {
    use csv::WriterBuilder;

    let mut writer = WriterBuilder::new()
        .has_headers(true)
        .double_quote(true)
        .flexible(false)
        .from_writer(std::io::stdout());

    let record_length = headers.len();

    writer.write_record(headers.clone())?;

    for row in rows {
        let in_row = SearchEntry::construct(row);
        let mut out_row: Vec<String> = Vec::with_capacity(record_length);

        for (i, key) in headers.iter().enumerate() {
            let value = match in_row.attrs.get(key) {
                None => String::new(),
                Some(values) => values.join(COMMA),
            };
            out_row.insert(i, value);
        }

        writer.write_record(out_row)?;
    }

    writer.flush()?;
    Ok(())
}
```

It really was quick, not so very dirty, and honestly easy to extend (I added JSON output using the `serde_json` crate). Was it easier than a shell script? Yes, given the specific complexity the script result would have been pretty much _write-only_ whereas the Rust code is eminently readable.