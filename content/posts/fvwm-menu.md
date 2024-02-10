---
title: 'FVWM Menus'
date: 2023-11-23T19:46:27-05:00
draft: false
tags: ["rust", "desktop", "programming"]
slug: "fvwm-menu"
---


# Journey for fvwm menu generator

- [X] Read $HOME/.local/share/applications

- [X] List all files with .desktop ext (kind of)

- [X] read each file

- [ ] build structs for menus

- [ ] build menu categories

- [ ] print menu

I want the location of each application directory of $HOME. I thought I would use some crate or standard library method like get_home_dir or something like that, then concatinate that to the “.local/share/applications” string. But it turned out to be less then easy. So I’ll just start with hardcoding the path, then when I understand how to use some of the libraries I’ll revisit this.

There is a std::env::home_dir function, but it's deprecated since 1.29, so this can wait.

Now that I have my path, I need to list all the files that have an extention of .desktop.

I can list the files in $HOME/.local/share/applications, but I need to filter on .desktop. That seems to be a little complicated. For now, I may just iterate over the list and check if "desktop" is in the file name.

The `.read_dir().expect("read_dir call failed")` can return an error. I should probably setup the `list_app_dir` function to return a Result type and let the caller handle the error.

Actually, I want the `list_app_dir` funtion to return a vec of Paths. 

```rust
    let files = list_app_dir(&app_dir);
    for name in files.iter() {
        println!("{:?}", name.path());
    }
```

The `name.path()` prints the entire path. 

```bash
"/home/rgeorgia/.local/share/applications/rustrover.desktop"
```

Now I need to read each file, place content into a struct, then return a vector of those structs.

## Reading file content

In my main I have this.

```rust
fn main() {

    let app_dir = Path::new("/home/rgeorgia/.local/share/applications");

    let files = list_app_dir(&app_dir);
    for name in files.iter() {
        let f = name.path().to_str().unwrap().trim();
        read_file_to_struct(f.clone()).expect("Error reading file");
    }
}
```

## temporary value dropped while borrowed Error

The read_file_to_struct function read the contents and, for now, pushes to a vec. I'll change that to read to a vec of structs. But for now I am getting an error.

```bash
❯ cargo run
   Compiling fvwm-menu v0.1.0 (/home/rgeorgia/workspace/rust_projects/fvwm-menu)
error[E0716]: temporary value dropped while borrowed
  --> src/main.rs:22:17
   |
22 |         let f = name.path().to_str().unwrap().trim();
   |                 ^^^^^^^^^^^                         - temporary value is freed at the end of this statement
   |                 |
   |                 creates a temporary value which is freed while still in use
23 |         read_file_to_struct(f.clone()).expect("Error reading file");
   |                             - borrow later used here
   |
   = note: consider using a `let` binding to create a longer lived value

For more information about this error, try `rustc --explain E0716`.
error: could not compile `fvwm-menu` (bin "fvwm-menu") due to previous error

```

So confusing. I really have no idea what the heck is going on. Did the `rustc --explain E0716` and read, reread and re-reread the contents. No light. No revelation. No ah ha. Just dark confusion.

I changed the for loop to this,

```rust
for name in files.iter() {
    let f = name.path();
    read_file_to_struct(f).expect("Error reading file");
}
```

And the `read_file_to_struct` function signature to this,

```rust
fn read_file_to_struct(file_to_read: PathBuf) -> std::io::Result<()> {
    let mut file = match File::open(file_to_read.to_str().unwrap().trim()) {
        Ok(file) => file,
        Err(error) => {
            match error.kind() {
                std::io::ErrorKind::NotFound => {
                    println!("File not found");
                    return Ok(());
                }
                _ => return Err(error),
            }
        }
    };
    let mut contents = Vec::new();
    file.read_to_end(&mut contents)?;

    println!("File contents: {:?}", contents);

    Ok(())
}
```

That "works," but of course it prints out a bunch of u8 characters. 

### Stop being fancy

Okay... time to stop being fancy. I'll just read the file, line by line and print it out. Later I'll populate the struct. But for now, this works.

```rust
fn read_file_to_struct(file_to_read: PathBuf) -> std::io::Result<()> {
    let file = File::open(file_to_read.to_str().unwrap().trim())?;
    let reader = BufReader::new(file);

    for line in reader.lines() {
        let line = line?;
        println!("{}", line);
    }

    Ok(())
}
```
Well... time to commit and push.

Desktop Entry
-------------

Now I want to read the file contents, line by line, into a struct to represent the ``[Desktop Entry]`` stanza. I decided to go with the following fields.

```rust
#[derive(Deserialize, Debug)]
pub struct DesktopEntry {
    pub name: String,
    pub app_type: String,
    pub icon: String, // Path
    pub exec: String, // Path
    pub comment: String,
    pub categories: Vec<String>,
    pub terminal: bool,
}
```

Now, when I look at the `.desktop` files, some of them have multiple comment lines, mulitple names, etc. I really just want the first one.

For example gparted.desktop has this,

```bash
[Desktop Entry]
Name=GParted
Name[ar]=مُقسِّم‌ج
Name[bg]=GParted
Name[br]=GParted
Name[bs]=GParted
<snip>
Comment=Create, reorganize, and delete partitions
Comment[ar]=أنشئ الأقسام ونظّمها واحذفها
Comment[bg]=Създаване, подреждане и изтриване на дялове
Comment[br]=Krouiñ, adaozañ ha dilemel ar parzhadoù
Comment[bs]=Kreiranje, reorganiziranje i brisanje particija
Comment[ca]=Creeu, reorganitzeu i suprimiu particions
```

To me, the `.desktop` files look like toml files. I'll try to use the toml crate to unmarshall the files into the DesktopEntry struct. Hopefully I can avoid a lot of manual checking of values.

Did as search for `toml` and rust from google and that lead me to the toml crate. Also, found this article that simplified a few things. [Rust Load a TOML File](https://codingpackets.com/blog/rust-load-a-toml-file/)

Ok, let's read about reading toml files into a struct.