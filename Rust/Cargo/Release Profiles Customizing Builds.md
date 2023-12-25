# Release Profiles: Customizing Builds

**release profiles:** predefined and customizable profiles with different configurations that allow a programmer to have more control over various options for compiling code

Each profile is configured independently of the others

Cargo has two main profiles

1. The `dev` profile Cargo uses when you run `cargo build` and 
    1. Good defaults for development
2. The `release` profile Cargo uses when you run `cargo build --release`
    1. Good defaults for release builds

Adding `[profile.*]` sections for any profile you want to customize, you override any subset of the default settings 

- For example, here are the default values for the `opt-level` setting for the `dev` and `release` profiles:

**Filename:** Cargo.toml

```rust
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` setting controls the number of optimizations Rust will apply to your code, with a range of 0 to 3

- Applying more optimizations extends compiling time
-