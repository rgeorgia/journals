---
title: "No Secret cat - nscat"
date: 2022-05-30T12:40:19-04:00
draft: true
categories: [posts]
tags: [rust, programming]

---

**As a** user **I want** a command-line program **that** displays the content of a file, but removes any 'secret' or sensitive information, like passwords or API keys.

## Use Case

While troubleshooting problems on a server, often the sysadmin in sharing their screen. There often comes a time when someone says, "What's in that .env file?". If the SA cats the file, there is a chance that any secrets in that file will be revealed to anyone viewing the shared screen. It would be nice if there was a program that would 'cat' the file but remove any sensitive information. 

## Slow and (not so) Easy

Because Rust has such a steep learning curve, and for a simpleton like myself, I need to go slow and easy.

### Step One

Read a file and print out each line.

1. Use a command line argument to pass in the name of a file to read.
2. Read the file into a variable.
3. Iterate over the variable, printing out each line.

### Step Two

Add a list of words to search for. For example, password, passwd, `_pass`, `_pw`, key...

