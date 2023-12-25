# Advanced Functions and Closures

## Function Pointers

We talked about how to pass closures to functions; you can also pass regular functions to functions

- Useful when you want to pass a function you’ve already defined rather than defining a new closure

Functions coerce to the type `fn` (with a lowercase f) 

- Not to be confused with the `Fn` closure trait

The `fn` type is called a *function pointer*

- Passing functions with function pointers will allow you to use functions as arguments to other functions

Syntax for specifying that a parameter is a function pointer is similar to that of closures:

- Defined a function `add_one` that adds one to its parameter
- The function `do_twice` takes two parameters:
    - A function pointer to any function that takes an `i32` parameter and returns an `i32`, and one `i32 value`
- The `do_twice` function calls the function `f` twice, passing it the `arg` value, then adds the two function call results together
- `main` function calls `do_twice` with the arguments `add_one` and `5`

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

This code prints `The answer is: 12`

- We specify that the parameter `f` in `do_twice` is an `fn` that takes one parameter of type `i32` and returns an `i32`
- We can then call `f` in the body of `do_twice`
- In `main`, we can pass the function name `add_one` as the first argument to `do_twice`

Unlike closures, `fn` is a type rather than a trait

- So we specify `fn` as the parameter type directly rather than declaring a generic type parameter with one of the `Fn` traits as a trait bound

Function pointers implement all three of the closure traits 

- (`Fn`, `FnMut`, and `FnOnce`), meaning you can always pass a function pointer as an argument for a function that expects a closure

It’s best to write functions using a generic type and one of the closure traits so your functions can accept either functions or closures

- One example of where you would want to only accept `fn` and not closures is when interfacing with external code that doesn’t have closures
- C functions can accept functions as arguments, but C doesn’t have closures

****************Example:**************** where you could use either a closure defined inline or a named function

- Use of the `map` method provided by the `Iterator` trait in the standard library

To use the `map` function to turn a vector of numbers into a vector of strings, we could:

- use a closure
- or we could name a function as the argument to `map` instead of the closure

```rust
let list_of_numbers = vec![1, 2, 3];

let list_of_strings: Vec<String> =
    list_of_numbers.iter().map(|i| i.to_string()).collect();
// -- OR --
let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(ToString::to_string).collect();
```

**Note:** We must use the fully qualified syntax

- Here, we’re using the `to_string` function defined in the `ToString` trait, which the standard library has implemented for any type that implements `Display`

**Note:** The name of each enum variant that we define also becomes an initializer function

- We can use these initializer functions as function pointers that implement the closure traits
- Means we can specify the initializer functions as arguments for methods that take closures

```rust
enum Status {
    Value(u32),
    Stop,
}

let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
```

- Here we create `Status::Value` instances using each `u32` value in the range that `map` is called on by using the initializer function of `Status::Value`

Some people prefer this style, and some people prefer to use closures: they compile to the same code

## Returning Closures

Closures are represented by traits, which means you can’t return closures directly

- You can instead use the concrete type that implements the trait as the return value of the function
- However, you can’t do that with closures because they don’t have a concrete type that is returnable
- For example: not allowed to use the function pointer `fn` as a return type

```rust
THIS WON'T WORK!
fn returns_closure() -> dyn Fn(i32) -> i32 { // error: return type cannot have an unboxed trait object
    |x| x + 1
}
```

- The error references the `Sized` trait again!

Problem: Rust doesn’t know how much space it will need to store the closure

Solution: We can use a trait object

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```