# Defining and Instantiating Structs

Structs are similar to tuples in that both hold multiple related values.

- Like tuples, the pieces of a struct can be different types
- Unlike with tuples, in a struct you’ll name each piece of data so it’s clear what the values mean

To define a struct, we enter the keyword `struct` and name the entire struct

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

To use a struct after we’ve defined it, we create an *instance* of that struct by specifying concrete values for each of the fields

We create an instance by stating the name of the struct and then add curly brackets containing *key: value* pairs

- **keys** are the names of the fields and the **values** are the data we want to store in those fields.
- We don’t have to specify the fields in the same order in which we declared them in the struct.

```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
}
```

To get a specific value from a struct, we use dot notation.

- For example, to access this user’s email we use `user1.email`
- If the instance is mutable, we can change a value by using the dot notation and assigning into a particular field

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

Note that the **entire instance** **must** be **mutable**

Rust doesn’t allow us to mark only certain fields as mutable

As with any expression, we can construct a new instance of the struct as the last expression in the function body to implicitly return that new instance. ei, no return

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
```

**Problem:** having to repeat the `email`and `username` field names and variables is a bit tedious.

### Using the Field Init Shorthand

Because the parameter names and the struct field names are exactly the same we can use the *field init shorthand* syntax to rewrite `build_user` so it behaves exactly the same but doesn’t have the repetition of `username` and `email`

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```

### Creating Instances from Other Instances with Struct Update Syntax

It’s often useful to create a new instance of a struct that includes most of the values from another instance, but changes some. You can do this using *struct update syntax*

```rust
fn main() {
    // --snip--
    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"), // Now user2 takes ownership
        sign_in_count: user1.sign_in_count,         // of email
    };
}
```

Using the struct update syntax `..` we can ahieve the same effect with less code

```rust
fn main() {
    // --snip--
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

The code in also creates an instance in `user2` that has a different value for `email` but has the same values for the `username`, `active`, and `sign_in_count` fields from `user1`

- The `..user1` must come **last** to specify that any remaining fields should get their values from the corresponding fields in `user1`

Note that the struct update syntax uses `=` like an assignment; this is because it moves the data

- we can no longer use `user1` as a whole after creating `user2` because the `String` in the `username` field of `user1` was moved into `user2`

### Using Tuple Structs Without Named Fields to Create Different Types

Rust also supports structs that look similar to tuples, called ***tuple structs***

- Have the added meaning the struct name provides but don’t have names associated with their fields
- Just have the types of the fields

Useful when you want to give the whole tuple a name and make the tuple a different type from other tuples, and when naming each field as in a regular struct would be verbose or redundant.

To define a tuple struct, start with the `struct` keyword and the struct name followed by the types in the tuple.

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

Each struct you define is its own type, even though the fields within the struct might have the same types

- Note that the `black` and `origin` values are **different types** because they’re instances of different tuple structs

You can use a `.` followed by the index to access an individual value

### Unit-Like Structs Without Any Fields

You can also define structs that don’t have any fields

These are called *unit-like structs* because they behave similarly to `()`, the unit type

Can be useful when you need to implement a trait on some type but don’t have any data that you want to store in the type itself

```rust
struct AlwaysEqual;
fn main() {
    let subject = AlwaysEqual;
}
```

- No need for curly brackets or parentheses
- we can get an instance of `AlwaysEqual` in the `subject`variable in a similar way: using the name we defined, without any curly brackets or parentheses.

## Traits

```rust
WON'T WORK!
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

 // Err `Rectangle` doesn't implement `std::fmt::Display`
 println!("rect1 is {}", rect1); 
}
```

The primitive types we’ve seen implement `Display` by default

- There’s only one way you’d want to show a `1`or any other primitive type to a user.

But with structs, you must implement `Display` for `println!` to work