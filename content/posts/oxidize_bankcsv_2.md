---
title: "Reading CSV file with Rust – Part 2"
date: 2023-11-23T19:46:27-05:00
draft: true
categories: ["posts"]
tags: ["rust", "desktop", "programming"]
slug: "oxidize2"
---


Okay, so I need to take a step back. 

## Things to keep in mind

- Try to optimize too early. Like moving code to their own file or package.
- Don't worry about being too DRY.
- Just getting it working.
- Do not be afraid to throw code away. i.e. keeping commented code because you are afraid that what you are doing now won't work and you don't remember where you got the last "working" code.
- Who owns the data?
- Passing by reference or by value?
- Is it mutable?

## Problem Re-statement

I have a csv file from my bank and I want to a payee column to contain **just** a payee name, that way I can make the spreadsheet more readable.

It has been a long time since I've visited this, and I've learned just a little more, I am going to rip everything out of main.rs and just start over.

## Starting over

First, I need the path to the csv file. For me it's in a data directory. Make sure this directory or it's content are in my .gitignore file. I don't want my banking info on github.

### Path to file

Do I add a cargo package to handle CLI arguments or just hard code the path? First, I will hard code the path name just to get things working. After things work I want to refactore to allow the a CLI path to file argument.

I will use `std::path::Path` to create a reference to a Path object. This will give me some flexability when working with the file.

```rust
use std::path::Path;

fn main() {
    let csv_file = Path::new("bankcsv/data/Checking1.csv");

    println!("{}", csv_file.to_string_lossy());
}
```
Cargo run just to make sure it works as expected.

```bash
cargo run
   Compiling bankcsv v0.1.0 (Oxidize-the-boring-stuff/bankcsv)
    Finished dev [unoptimized + debuginfo] target(s) in 1.08s
     Running `target/debug/bankcsv`
bankcsv/data/Checking1.csv
```

### Reading the file

Now I want to read the file. Since I know that this is a csv file I figure there **must** be a crate already out there for me to use. 
Go to [crates.io](https://crates.io) and search for csv. One thing I do when deciding on a crate is to look at the number of downloads and recent downloads, but especially the activity. If the last time it was updated was 5 years ago, I'll probably pass on that crate. I will go with the first one on the list, since it is written by BurntSushi, and who hasn't heard of him? Not to mention the issues and pull requests look like they do recieve attention.

I add the latest version of csv to my Cargo.toml file. I also head over to the [csv](https://crates.io/crates/csv) crates.io page. I also notice a really nice [csv cookbook](https://docs.rs/csv/1.0.0/csv/cookbook/index.html)

```rust
[dependencies]
csv = "1.1.6"
```

Okay, what do I want to do? I want to read each line of the file, then just print each 'field' in each row. That's it. Later I'll create a struct and add the values.

From the [BurntSushi/rust-csv](https://github.com/BurntSushi/rust-csv/tree/master/examples) site I find two examples that can get me going.

- [tutorial-read-01.rs](https://github.com/BurntSushi/rust-csv/blob/master/examples/tutorial-read-01.rs) 
- [tutorial-read-serde-01.rs](https://github.com/BurntSushi/rust-csv/blob/master/examples/tutorial-read-serde-01.rs)


