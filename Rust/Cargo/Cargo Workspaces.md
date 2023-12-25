# Cargo Workspaces

As your project develops, you might find that the library crate continues to get bigger and you want to split your package further into multiple library crates

*workspaces* that can help manage multiple related packages that are developed in tandem

## Creating a workspace

**workspace:** a set of packages that share the same *Cargo.lock* and output directory

**Example:** Workspace containing a binary and two libraries

- Binary will provide the main functionality, will depend on the two libraries
    - One library will provide an `add_one` function, and a second library an `add_two` function.
- These three crates will be part of the same workspace.

```bash
$ mkdir add
$ cd add
```

Next, in the *add* directory, we create the *Cargo.toml* file that will configure the entire workspace

- This file won’t have a `[package]` section
- Instead, it will start with a `[workspace]` section that will allow us to add members to the workspace by specifying the path to the package with our binary crate;
    - in this case, that path is *adder*

```toml
[workspace]
members = [
    "adder",
]
```

Next, we’ll create the `adder` binary crate by running `cargo new` within the *add* directory:

```bash
$ cargo new adder
```

Dir structure should look like this

```
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

- Workspace has one *target* directory at the top level that the compiled artifacts will be placed into
    - `adder` package doesn’t have its own *target* directory
    - Even if we were to run `cargo build` from inside the *adder* directory, the compiled artifacts would still end up in *add/target* rather than *add/adder/target*
- Cargo structures the *target* directory in a workspace like this because the crates in a workspace are meant to depend on each other.
    - If each crate had its own *target* directory, each crate would have to recompile each of the other crates in the workspace to place the artifacts in its own *target* directory
    - By sharing one *target* directory, the crates can avoid unnecessary rebuilding.

## Creating the Second Package in the Workspace

Next, let’s create another member package in the workspace `add_one`
Change the top-level *Cargo.toml* to specify the *add_one* path in the `members` list

```toml
[workspace]

members = [
    "adder",
    "add_one",
]
```

Then generate a new library crate named `add_one`

```toml
$ cargo new add_one --lib
```

Dir structure should look like this

```
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Implement `add_one` function:

**Filename:** add_one/src/lib.rs

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

Now we can have the `adder` package with our binary depend on the `add_one` package that has our library. 
First, we’ll need to add a path dependency on `add_one` to *adder/Cargo.toml*.

**Filename:** adder/Cargo.toml

```toml
[dependencies]
add_one = { path = "../add_one" }
```

Cargo doesn’t assume that crates in a workspace will depend on each other, so we need to be explicit about the dependency relationships

Next, let’s use the `add_one` function (from the `add_one` crate) in the `adder` crate

**Filename:** adder/src/main.rs

```rust
use add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {num} plus one is {}!", add_one::add_one(num));
}
```

To run the binary crate from the *add* directory, we can specify which package in the workspace we want to run by using the `-p` argument and the package name with `cargo run`:

```bash
$ cargo run -p adder
```

## Depending on an External Package in Workspace

**Note:** workspace has only one *Cargo.lock* file at the top level rather than having a *Cargo.lock* in each crate’s directory

- ensures that all crates are using the same version of all dependencies

If we add the `rand` package to the *adder/Cargo.toml* and *add_one/Cargo.toml* files, Cargo will resolve both of those to one version of `rand` and record that in the one *Cargo.lock*

- Making all crates in the workspace use the same dependencies means the crates will always be compatible with each other

**Filename:** add_one/Cargo.toml

```toml
[dependencies]
rand = "0.8.5"
```

We can now add `use rand;` to the *add_one/src/lib.rs* file, and building the whole workspace by running `cargo build` in the *add* directory will bring in and compile the `rand` crate.

The top-level *Cargo.lock* now contains information about the dependency of `add_one` on `rand`. However, even though `rand` is used somewhere in the workspace, we can’t use it in other crates in the workspace unless we add `rand` to their *Cargo.toml* files as well.