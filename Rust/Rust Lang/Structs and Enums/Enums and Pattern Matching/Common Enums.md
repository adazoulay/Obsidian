# Common Enums

### **`Option<T>`**: Represents an optional value.

- Two variants: **`Some(T)`** for a value of type **`T`**, and **`None`** for the absence of a value.
- **`Option`**is often used in cases where a value may or may not be present, such as when looking up a key in a hash map or accessing an element in a vector by index.

```rust
pub enum Option<T> {
    Some(T),
    None,
}
```

- Common Methods:
    - **`is_some()`**: Returns **`true`** if the **`Option`** is a **`Some`** variant, and **`false`** otherwise.
    - **`is_none()`**: Returns **`true`** if the **`Option`** is **`None`**, and **`false`** otherwise.
    - **`unwrap()`**: Extracts the value inside **`Some`**, or panics if the **`Option`** is **`None`**.
    - **`unwrap_or(default)`**: Extracts the value inside **`Some`**, or returns the given **`default`** if the **`Option`** is **`None`.**
    - **`as_ref`**: ****Allows to take a reference to the value inside an **`Option<T>`** without consuming it; ie. convert an **`Option<T>`** into an **`Option<&T>`**
    - `**take()**`: ****Allows you to take the value out of an **`Option`** and replace it with **`None`**
    - `**map()**`: Transforms the value inside **`Some`** using a closure, and it returns a new **`Option`** that contains the transformed value
- Implements `**Copy`** trait

### **`Result<T, E>`**: Represents the result of a computation that may fail.

- Two variants: **`Ok(T)`** for a successful result with a value of type **`T`**, and **`Err(E)`** for an error or failure with an error value of type **`E`**.
- **`Result`** is frequently used for error handling, especially when dealing with I/O, parsing data, or making network requests.

```rust
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

- Common Methods:
    - **`is_ok()`**: Returns **`true`** if the **`Result`** is an **`Ok`** variant, and **`false`** otherwise.
    - **`is_err()`**: Returns **`true`** if the **`Result`** is an **`Err`** variant, and **`false`** otherwise.
    - **`unwrap()`**: Extracts the value inside **`Ok`**, or panics if the **`Result`** is an **`Err`** variant.
    - **`unwrap_err()`**: Extracts the error inside **`Err`**, or panics if the **`Result`** is an **`Ok`** variant.
    - **`unwrap_or(default)`**: Extracts the value inside **`Ok`**, or returns the given **`default`** if the **`Result`** is an **`Err`** variant.
    - **`unwrap_or_else(default)`:** allows us to define some custom, non-`panic!` error handling.
    - **`as_ref`:**  Allows to take a reference to the value inside an **`Result<T, E>`** without consuming it; ie. convert an **`Result<T, E>`** into  **`Result<&T, &E>`.**

### **`std::cmp::Ordering`**:  Represents the result of a comparison between two values.

- Three variants: **`Less`**, **`Equal`**, and **`Greater`**.
- It's used in sorting and ordering operations and is the return type for the **`cmp`** method of the **`Ord`** trait.

```rust
pub enum Ordering {
    Less,
    Equal,
    Greater,
}
```

### **`std::io::ErrorKind`**: Represents different kinds of I/O errors

- Can occur while working with files, sockets, and other I/O operations
- Used in conjunction with the **`std::io::Error`**type to provide detailed error information.

```rust
pub enum ErrorKind {
    NotFound,
    PermissionDenied,
    ConnectionRefused,
    // ... other variants ...
}
```

### **`std::num::ParseIntError`**: Represents errors that can occur when parsing integers from strings

- Note that **`ParseIntError`** is a struct, not an enum. Its fields store the error kind and error message.

```rust
pub struct ParseIntError {
    kind: IntErrorKind,
}

pub(crate) enum IntErrorKind {
    Empty,
    InvalidDigit,
    Overflow,
    Underflow,
}
```

### **`std::net::IpAddr`**: Represents an IP address

- Two variants: **`V4`** for IPv4 addresses and **`V6`** for IPv6 addresses.
- Used when working with networking code, sockets, and IP address manipulation.

```rust
pub enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```