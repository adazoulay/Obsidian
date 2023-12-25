# Packages and Crates

### Crates

A *crate* is the smallest amount of code that the Rust compiler considers at a time

- Crates can contain modules, and the modules may be defined in other files that get compiled with the crate

if you run `rustc` rather than `cargo` and pass a single source code file, the compiler considers that file to be a crate

A crate can come in one of two forms: 

- a binary crate
    - *Binary crates* are programs you can compile to an executable that you can run
    - Each must have a function called `main` that defines what happens when the executable runs
        - Ex: a command-line program or a server
- a library crate
    - *Library crates* don’t have a `main` function, and they don’t compile to an executable
    - Instead, they define functionality intended to be shared with multiple projects
        - Ex: the `rand` crate provides functionality that generates random numbers.

Most of the time when Rustaceans say “crate”, they mean library crate, and they use “crate” interchangeably with the general programming concept of a “library"

The *crate root* is a source file that the Rust compiler starts from and makes up the root module of your crate

### Packages

A *package* is a bundle of one or more crates that provides a set of functionality

A package contains a *Cargo.toml* file that describes how to build those crates

- Cargo is actually a package that contains the binary crate for the command-line tool you’ve been using to build your code
- The Cargo package also contains a library crate that the binary crate depends on

A package can contain as many binary crates as you like, but at most only one library crate

A package must contain at least one crate, whether that’s a library or binary crate.

Here is what happens when we create a new package

```rust
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

- In *Cargo.toml* note there’s no mention of *src/main.rs*
    - Cargo follows a convention that *src/main.rs* is the crate root of a binary crate with the same name as the package
- Cargo knows that if the package directory contains *src/lib.rs*, the package contains a library crate with the same name as the package, and *src/lib.rs*
 is its crate root
- Cargo passes the crate root files to `rustc` to build the library or binary

If a package contains *src/main.rs* and *src/lib.rs*, it has two crates: a binary and a library, both with the same name as the package