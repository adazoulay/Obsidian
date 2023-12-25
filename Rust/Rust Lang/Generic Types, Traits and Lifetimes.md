# Generic Types, Traits and Lifetimes

[[Generic Data Types]]

[[Traits  Defining Shared Behavior]]

[[Lifetimes  Validating References]]

[[Common Attributes and Traits]]

## Generic Type Parameters, Traits, Bounds and Lifetimes Together

********Example:******** Syntax of specifying generic type parameters, trait bounds, and lifetimes all in one function

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

- `longest` function: returns the longer of two string slices
- Now has an extra parameter named `ann` of the generic type `T`, which can be filled in by any type that implements the `Display` trait as specified by the `where` clause
    - `ann` will be printed using `{}`, which is why the `Display` trait bound is necessary.
- Because lifetimes are a type of generic:
    - The declarations of the lifetime parameter `'a` and the generic type parameter `T` go in the same list inside the angle brackets after the function name.