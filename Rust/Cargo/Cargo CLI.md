# Cargo CLI

V****************************ersioning****************************

- `cargo --version`
    - Returns current cargo Version
- Updating:
    - `rustup update`
    - `cargo update`
- Switching:
    - `rustup default nightly`
    - `rustup default stable`
    

**Creating a new project**

- `cargo new hello_cargo` + Optional arg
    - Optional arg:
        - `--lib` for library
        - `--bin` for binary (same as no arg)
    - Creates a new directory and project called *hello_cargo*
    - Generates two files and 1 dir
        - *Cargo.toml* : Config file
        - *.gitignore*
        - *src* directory
            - [*main.rs*](http://main.rs) file inside
        

**Ways to build**

- `cargo build`
    - creates an executable file in *target/debug/hello_cargo*
    - creates file: *Cargo.lock (k*eeps track of exact versions of dependencies in project)
    - To run: `./target/debug/hello_cargo`
- `cargo run` (more convenient and used)
    - Builds and runs program
    - Can add `-p` flag to target package in workspace. Ex: `cargo run -p adder`
- `cargo check`
    - Quickly checks your code to make sure it compiles but doesn’t produce an executable
- `cargo build --release`
    - Used to build for release, compiles with optimizations
    - Creates an executable in *target/release*

**Profiling**

- `time cargo run`

**********************************Managing packages**********************************

- `cargo clean`
    - Remove artifacts from the target directory that Cargo has generated in the past.
    - With no options, `cargo clean` will delete the entire target directory.
- `cargo update`
    - ignore the *Cargo.lock* and figure out all the latest crate versions that fit specifications in *Cargo.toml*
- `cargo doc --open`
    - Builds documentation provided by all local dependencies and opens in browser
- `cargo vendor`
    - Downloads and stores all the dependencies required by Rust project locally in a separate directory.
    
    ```rust
    [dependencies]
    dependency-name = { path = "vendor/dependency-name" }
    ```
    

******************************************Installing dependencies******************************************

- `cargo install`
    - Allows you to install and use binary crates locally
    - **Not** intended to replace system packages;
    - Can only install packages that have binary targets