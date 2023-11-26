---
title: "Reading CSV file with Rust"
date: 2023-11-23T19:46:27-05:00
draft: false
categories: ["posts"]
tags: ["rust", "desktop", "programming"]
slug: "oxidize"
---

<img src="/images/rust_crab.png" alt="Rusty" title="Rusty" width="300px">

## Purpose

Learn to read and parse a csv file with Rust.

## Problem Statement

 I have a csv file from my bank and I want to a payee column to contain **just** a payee name, that way I can make the spreadsheet more readable.

## End Result

A csv file formatted as `date, amount, check number, payee, category`

Example:
```bash
10/21/2021,-14.44,,"SHAWS","FOOD"
10/24/2021,-100.00,1234,"ST LUKE","GIVING"
```

## Steps

- Read the csv file downloaded from my bank.
- Add a payee column by extracing the payee from all the extra verbiage of the original download. For example adding "FOOD LION" from `"PURCHASE AUTHORIZED ON 10/14 FOOD LION #2856 NOWHERE USA P00000000123456789 CARD 4242"`. 
- Add category field based on the extracted payee
- Print out to csv file, create or append. 

## Percolating Ideas

- Save data to sqlite3 db
- What do I need to capture to prevent duplicate entries into the db?
- Provide options to generate reports

Need to fill in the journey up to this point

## The Journey

I did have two structs, one for Checking and one for Credit Card, but they had identical fields and field types. The only difference was the Checking struct had _check_number_ while CreditCard had _other_, but both ware Strings. So, I decided to create one struct called TransactionEvent.

```rust
#[derive(Debug, Deserialize)]
#[serde(rename_all = "PascalCase")]
struct TransactionEvent {
    date: String,
    amount: Option<f64>,
    star: String,
    check_number: String,
    payee: String,
}
```

Now I read the file into a String, then use `csv::ReaderBuilder::new()` to create a new reader to allow me to store the file contents in a struct. I just want to print out what I have.

```rust
for result in rdr.deserialize() {
    let record: TransactionEvent = result?;
    println!("{}", record.date);
    println!("{}", record.payee);
    println!("{}", record.amount);
    println!("{}\n", record.check_number);
}
```

But this does not work. I get a compile error.

```bash
Error[E0277]: `std::option::Option<f64>` doesn't implement `std::fmt::Display`
  --> src/main.rs:59:24
   |
59 |         println!("{}", record.amount);
   |                        ^^^^^^^^^^^^^ `std::option::Option<f64>` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `std::option::Option<f64>`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)
```

Making the suggested adjustment, i.e. adding `{:?}` I now get this output.

```bash
07/26/2021
LA FUENTE MEXICAN GRILL
Some(-33.47)
```

Not quiet what I want, but better. Since `record.amount` is an Option type you need to `unwrap()` the result. I change my println to the following:

```rust
println!("{}", record.amount.unwrap());
```

### Things can go wrong - questions to ask yourself and address

- What if there's a non f64 'character' is in the amount column? 

For example a `"*"` instead of a `"-33.90"`?

The code will compile and run, but ends (crashes?) with the following error.

```bash
CSV deserialize error: record 158 (line: 159, byte: 10091): field 1: invalid float literal
```

- How do I skip the "star" field? 
- Is there a way to read the file, use a new reader and ignore as specific column?
