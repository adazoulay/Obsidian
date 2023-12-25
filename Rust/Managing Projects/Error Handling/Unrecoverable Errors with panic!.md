# Unrecoverable Errors with panic!

Sometimes, bad things happen in your code, and there’s nothing you can do about it 
In these cases, Rust has the `panic!` macro

Two ways to cause a panic in practice

- Taking an action that causes our code to panic (such as accessing an array past the end)
- Explicitly calling the `panic!`macro

Panics will print a failure message, unwind, clean up the stack, and quit

## Unwinding the Stack or Aborting in Response to a Panic

By default, when a panic occurs, the program starts *unwinding*, 

- Rust walks back up the stack and cleans up the data from each function it encounters
    - This walking back and cleanup is a lot of work
    - Rust, therefore, allows you to choose the alternative of immediately *aborting*, which ends the program without cleaning up
- Memory the program was using will then need to be cleaned up by the operating system
- To make the resulting binary as small as possible, you can switch from unwinding to aborting upon a panic
    - by adding `panic = 'abort'` to the appropriate `[profile]` sections in your *Cargo.toml*

```rust
[profile.release]
panic = 'abort'
```

****************Example:****************

```rust
fn main() {
    panic!("crash and burn");
}
// Output: thread 'main' panicked at 'crash and burn', src/main.rs:2:5
```

- In this case, the line indicated is part of our code
- `panic!` call might be in code that our code calls, and the filename and line number reported by the error message will be someone else’s code where the `panic!`
 macro is called, not the line of our code that eventually led to the `panic!`

We can use the **backtrace** of the functions the `panic!` call came from to figure out the part of our code that is causing the problem

****************Example****************: Array out of bounds

```rust
fn main() {
    let v = vec![1, 2, 3];
    v[99];
} // Output: thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
```

We can set the `RUST_BACKTRACE`environment variable to get a backtrace of exactly what happened to cause the error

- A *backtrace* is a list of all the functions that have been called to get to this point

```rust
In terminal
$ RUST_BACKTRACE=1 cargo run
```

In order to get backtraces, debug symbols must be enabled

- Enabled by default when using `cargo build` or `cargo run` without the `--release` flag,