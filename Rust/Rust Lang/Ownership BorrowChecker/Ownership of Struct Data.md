# Ownership of Struct Data

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

[[Defining and Instantiating Structs]]

In the `User` struct definition in  we used the owned `String` type rather than the `&str` string slice type.

- This is a deliberate choice because we want each instance of this struct to own all of its data and for that data to be valid for as long as the entire struct is valid.

It’s also possible for structs to store references to data owned by something else, but to do so requires the use of *lifetimes*

- Lifetimes ensure that the data referenced by a struct is valid for as long as the struct is.

Ex: Let’s say you try to store a reference in a struct without specifying lifetimes, like the following; this won’t work

```rust
THIS WON'T WORK!
struct User {
    active: bool,
    username: &str,
    email: &str,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        active: true,
        username: "someusername123",
        email: "someone@example.com",
        sign_in_count: 1,
    };
}
```

In **Chapter 10**, we’ll discuss how to fix these errors so you can store references in structs, but for now, we’ll fix errors like these using owned types like `String`
 instead of references like `&str.`