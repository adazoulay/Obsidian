# All Places Patterns Can Be Used

## `match` Arms

`match` allows you to compare a value against a series of patterns and then execute code based on which pattern matches

Formally, `match` expressions are defined as:

- The keyword `match`, a value to match on, and one or more match arms that consist of a pattern and an expression to run if the value matches that arm’s pattern

`match` expressions need to be *exhaustive*

- The particular pattern `_` will match anything, but it never binds to a variable

```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

**Example:** `match` expression that matches on an `Option<i32>` value in the variable `x`

```rust
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

## `if let` Expressions

`if let`: A shorter way to write the equivalent of a `match` that only matches one case

- Optionally: `if let` can have a corresponding `else` containing code to run if the pattern in the `if let` doesn’t match
    - It’s also possible to mix and match `if let`, `else if`, and `else if let`
        - Gives us more flexibility than a `match` expression in which we can express only one value to compare with the patterns
        - Doesn't require that the conditions in a series of `if let`, `else if`, `else if let` arms relate to each other

**Example:** This conditional structure lets us support complex requirements

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

`if let` can also introduce shadowed variables in the same way that `match` arms can

- The line `if let Ok(age) = age` introduces a new shadowed `age` variable that contains the value inside the `Ok` variant
- However, shadowed var need to be inside block. This won’t work: `let Ok(age) = age && age > 30`

Downside of using `if let` expressions is that the compiler doesn’t check for exhaustiveness

## `while let` Conditional Loops

`while let` conditional loop allows a `while` loop to run for as long as a pattern continues to match

**Example:** `while let` loop that uses a vector as a stack and prints the values in the vector in the opposite order in which they were pushed

```rust
let mut stack = Vec::new();

    stack.push(1);
    stack.push(2);
    stack.push(3);

    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
```

- Prints: 3, 2, 1
- `pop` method takes the last element out of the vector and returns `Some(value)`
    - If the vector is empty, `pop` returns `None`

## `for` Loops

In a `for` loop, the value that directly follows the keyword `for` is a pattern

- For example, In `for x in y`: The `x` is the pattern

****************Example:**************** Demonstrates how to use a pattern in a `for` loop to destructure, or break apart, a tuple as part of the `for` loop.

```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

- We adapt an iterator using the `enumerate` method so it produces a value and the index for that value, placed into a tuple

## `let` Statements

`let` is also a pattern

```rust
let x = 5;
// More formally:
let PATTERN = EXPRESSION;
```

Rust compares the expression against the pattern and assigns any names it finds

- So in the `let x = 5;` example, `x` is a pattern that means: “bind what matches here to the variable `x`.

To see the pattern matching aspect of `let` more clearly, consider the example below, which uses a pattern with `let` to destructure a tuple``

```rust
let (x, y, z) = (1, 2, 3);
```

Here, we match a tuple against a pattern

- Rust compares the value `(1, 2, 3)` to the pattern `(x, y, z)` and sees that the value matches the pattern
    - So Rust binds `1` to `x`, `2` to `y`, and `3` to `z`

If the number of elements in the pattern doesn’t match the number of elements in the tuple, the overall type won’t match

- We’ll get a compiler error:

```rust
let let (x, y) = (1, 2, 3); // error: mismatched types
```

To fix the error, we could ignore one or more of the values in the tuple using `_` or `..`

## Function Parameters

Function parameters can also be patterns

**Example:** The code below declares a function named `foo`: takes one parameter named `x` of type `i32`

```rust
fn foo(x: i32) { code goes here }
```

- Here, `x` is a pattern

As we did with `let`, we could match a tuple in a function’s arguments to the pattern:

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

- The values `&(3, 5)` match the pattern `&(x, y)`
    - `x` is the value `3` and `y` is the value `5`

**************Note:************** We can also use patterns in **closure parameter lists** in the same way as in function parameter lists, because closures are similar to functions