# Recoverable Errors with Result

Most errors aren’t serious enough to require the program to stop entirely

- Example: if you try to open a file and fails because the file doesn’t exist → create the file instead of terminating the process

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

- `T` represents the type of the value that will be returned in a success case within the `Ok` variant
- `E`represents the type of the error that will be returned in a failure case within the `Err` variant

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```

Return type of `File::open` is a `Result<T, E>` 

- Generic parameter `T` has been filled in by the implementation of `File::open` with the type of the success value, `std::fs::File`, which is a file handle
- Type of `E` used in the error value is `std::io::Error`

## Propagating Errors

P*ropagating:* When a function’s implementation calls something that might fail, instead of handling the error within the function itself, you can return the error to the calling code so that it can decide what to do

- Gives more control to the calling code

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

- Return type of `read_username_from_file`  :  `Result<String, io::Error>`
    - `Result<T, E>`
        - `T` type `String`
        - `E` type `io::Error`
- handle the `Result` returned by `File::open` with a `match`
    - `File::open` success case:
        - Mutable variable `username_file` takes value of pattern variable `file` and the function continues
    - `Err` case: instead of calling `panic!`
        - Use the `return` keyword to return early out of the function entirely
        - Pass error value from `File::open` to pattern variable `e` back to the calling code as this function’s error value.
- `read_to_string` method also returns a `Result`because it might fail even though `File::open` succeeded.
    - So we need another `match` to handle that `Result`

Up to the calling code to decide what to do with those values

- If the calling code gets an `Err` value, it could call `panic!` and crash the program,
- Or use a default username, or look up the username from somewhere other than a file, for example.

This pattern of propagating errors is so common in Rust that Rust provides the question mark operator `?` to make this easier

## `?` Operator: Shortcut for Propagating Errors

`?` placed after a `Result`value is defined to work in almost the same way as the `match` expressions we defined to handle the `Result` values

When you use the **`?`** operator on a **`Result`** value, it does the following:

1. If the **`Result`** is an **`Ok`** variant, it extracts the value contained within the **`Ok`** and returns it, the program continues
2. If the **`Result`** is an **`Err`** variant, it returns the error value immediately, propagating it up the call stack

****************Example:**************** Same functionality as code above, but using `?` operator

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

- If the value of the `Result<String, io::Error>` is an `Ok`
    - The value inside the `Ok` will get returned from this expression, and the program will continue.
    - The value is an `Err`, the `Err` will be returned from the whole function as if we had used the `return` keyword so the error value gets propagated to the calling code.

**Difference with Match**

difference between `match` expression and what the `?` operator

- Error values that have the `?` operator called on them go through the `from` function
    - Defined in the `From` trait in the standard library, used to convert values from one type into another
- When the `?` operator calls the `from` function, the error type received is converted into the error type defined in the return type of the current function
    - Useful when a function returns one error type to represent all the ways a function might fail, even if parts might fail for many different reasons

`?` operator can be chained to have even more concise syntax

```rust
...
fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();
    File::open("hello.txt")?.read_to_string(&mut username)?;   // chained ?
    Ok(username)
}
OR EVEN SHORTER
fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")  // using fs::read_to_string 
}
```

## Where The `?` Operator Can Be Used?

The **`?`** operator can be **used on** any expression that evaluates to a type that implements the **`std::ops::Try`** trait

- `Result` and `Option` type implements the `Try` trait

`?` operator can only be **used in** functions whose return type is `Result` or `Option` 

- (or another type that implements `FromResidual`)

So If the function is `void` or it’s return type doesn’t match the error or extracted value from `?` won’t work

```rust
THIS WON'T WORK!
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")?;
}
```

This code opens a file, which might fail

- The `?` operator follows the `Result` value returned by `File::open`, but this `main` function has the return type of `()`, not `Result`

Two options to solve error

1. Change the return type of your function to be compatible with the value you’re using the `?` operator on as long as you have no restrictions preventing that. 
2. Use a `match` or one of the `Result<T, E>` methods to handle the `Result<T, E>` in whatever way is appropriate

### Behaviour in Option

The behavior of the `?` operator when called on an `Option<T>` is similar to its behavior when called on a `Result<T, E>`

- If the value is `None`, the `None` will be returned early from the function at that point
- If the value is `Some`, the value inside the `Some` is the resulting value of the expression and the function continues

**Example:**

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

- This function returns `Option<char>` because it’s possible that there is a character there, but it’s also possible that there isn’t
- takes the `text` string slice argument and calls the `lines` method on it
    - returns an iterator over the lines in the string
- Calls `next` on the iterator to examine the first line and get the first value from the iteraton
    - If `text` is the empty string
        - Call to `next` will return `None`, in which case we use `?` to stop and return `None` from `last_char_of_first_line`.
    - If `text` is not the empty string
        - `next` will return a `Some` value containing a string slice of the first line in `text`
- `?` extracts the string slice, and we can call `chars` on that string slice to get an iterator of its characters
- `last` returns the last item in the iterator

### Converting between Option and Result types

- **Can** use the `?` operator on a `Result` in a function that returns `Result`, and you can use the `?` operator on an `Option` in a function that returns `Option`,
- **can’t** mix and match. The `?` operator won’t automatically convert a `Result` to an `Option` or vice versa
    - In those cases, you can use methods like the `ok` on `Result` or the `ok_or` method on `Option` to do the conversion explicitly

## Use in `main`

`main` can also return a `Result<(), E>`

Change the return type of `main` to be `Result<(), Box<dyn Error>>` and add a return value `Ok(())`

```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```

`Box<dyn Error>` type is a *trait object*

- Means “any kind of error