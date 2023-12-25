# Defining an Enum

Where structs give you a way of grouping together related fields and data, like a `Rectangle` with its `width` and `height`, enums give you a way of saying a value is one of a possible set of values.

We may want to say that `Rectangle`is one of a set of possible shapes that also includes `Circle` and `Triangle`
To do this, Rust allows us to encode these possibilities as an enum.

Example: Say we need to work with IP addresses. 

- Currently, two major standards are used for IP addresses: version four and version six.
- Because these are the only possibilities for an IP address that our program will come across, we can *enumerate* all possible variants
- Any IP address can be either a version four or a version six address, but not both at the same time
    - Makes the enum data structure appropriate because an enum value can only be one of its variants

We can express this by defining an `IpAddrKind` enumeration and listing the possible kinds an IP address can be, `V4` and `V6`

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

`IpAddrKind` is now a custom data type that we can use elsewhere in our code

## Enum Values

We can create instances of each of the two variants of `IpAddrKind` like so

```rust
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

Note that the variants of the enum are namespaced under its identifier, and we use a double colon `::` to separate the two. 

- This is useful because now both values `IpAddrKind::V4`and `IpAddrKind::V6` are of the same type: `IpAddrKind`

We can then, for instance, define a function that takes any `IpAddrKind`

```rust
fn route(ip_kind: IpAddrKind) {}
// We can call this function with either variant
route(IpAddrKind::V4);
route(IpAddrKind::V6);
```

More advantages

- At the moment we don’t have a way to store the actual IP address *data*
    - We only know what *kind* it is. Given that you just learned about structs in Chapter 5, you might be tempted to tackle this problem with structs as shown in Listing 6-1.
    - We could use a `struct` with a `IpAddrKind` to store kind and a `String` to store the IP address
- However, Enums allow us to store data directly

```rust
enum IpAddr {
        V4(u8, u8, u8, u8), // Can hold any type
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));
```

Advantages

- Representing the same concept using just an enum is more concise
    - Rather than an enum inside a struct, we can put data directly into each enum variant
- The name of each enum variant that we define also becomes a function that constructs an instance of the enum
    - `IpAddr::V4()`is a function call that takes a `String` argument and returns an instance of the `IpAddr`
    - Automatically get this constructor function defined as a result of defining the enum
- Each variant can have different types and amounts of associated data.
    - `V4` will always have four numeric components that will have values between 0 and 255.

Note: wanting to store IP addresses and encode which kind they are is so common that [the standard library has a definition we can use](https://doc.rust-lang.org/std/net/enum.IpAddr.html)

**Example** with many types/variants

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
------ EQUIVALENT ------
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

This enum has four variants with different types:

- `Quit` has no data associated with it at all.
- `Move` has named fields, like a struct does.
- `Write` includes a single `String`.
- `ChangeColor` includes three `i32` values.

If we used the different structs, each of which has its own type, we couldn’t as easily define a function to take any of these kinds of messages as we could with the `Message`
 enum

## Calling Methods on Enums

Just as we’re able to define methods on structs using `impl`, we’re also able to define methods on enums

```rust
impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }
...
    let m = Message::Write(String::from("hello"));
    m.call();
```

## The `Option` Enum and its Advantages over Null Values

`Option` another enum defined by the standard library

`Option` type encodes the very common scenario in which a value could be something or it could be nothing.

Example: if you request the first item in a non-empty list, you would get a value. If you request the first item in an empty list, you would get nothing.

- Expressing this concept in terms of the type system means the compiler can check whether you’ve handled all the cases you should be handling
- Can prevent bugs that are extremely common in other programming languages

### Null in Rust

Rust does **not** have ********Null********

Rust **does** have an `enum` that can encode the concept of a value being present or absent 

This enum is `Option<T>`, and it is [defined by the standard library](https://doc.rust-lang.org/std/option/enum.Option.html) as follows

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`**unwrap**` method allows you to retrieve the data from `Some(T)`

The `Option<T>` enum is so useful that it’s even included in the prelude

- You don’t need to bring it into scope explicitly

Its variants are also included in the prelude

- You can use `Some` and `None`directly without the `Option::` prefix

Node `<T>` is syntax for generics

```rust
let some_number = Some(5); // Type of some_number is Option<i32>
let some_char = Some('e'); // Type of some_char is Option<char>
let absent_number: Option<i32> = None; 
```

For `absent_number`, Rust requires us to annotate the overall `Option` type 

- The compiler can’t infer the type that the corresponding `Some` variant will hold by looking only at a `None`value.
- Here, we tell Rust that we mean for `absent_number` to be of type `Option<i32>`

In order to have a value that can possibly be null, you must explicitly opt in by making the type of that value `Option<T>`

Then, when you use that value, you are required to explicitly handle the case when the value is null. 

Everywhere that a value has a type that isn’t an `Option<T>`, you *can* safely assume that the value isn’t null.