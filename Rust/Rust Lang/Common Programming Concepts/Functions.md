# Functions

Rust uses *snake case* as convention

We define a function through `fn` keyword followed by function name and parenthesis

```rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

Functions are called by `func_name();`

Note: `another_function` defined *after* the `main` function in the source code

- Rust doesn’t care where you define your functions, only that they’re defined somewhere in a scope that can be seen by the caller.

## Parameters

We can define functions to have parameters

```rust
fn main() {
    another_function(5);
		a_third_function(20,'s');
}

fn another_function(x: i32) {
    println!("The value of x is: {x}");
}

fn a_third_function(value: i32, unit_label: char) {
    println!("The measurement is: {value}{unit_label}");
}
```

`another_function` has one parameter named x of type i32

**In function signatures, you *must* declare the type of each parameter**

This is a deliberate decision in Rust’s design.

- Requiring type annotations in function definitions means the compiler almost never needs you to use them elsewhere in the code to figure out what type you mean.

`a_third_function` has a parameter `value` of type `i32` and other `unit_label` of type `char`

- When defining multiple params, separate param declarations with commas

### Statements and Expressions

Function bodies are made up of a series of statements optionally ending in an expression.

- **Statements** are instructions that perform some action and do not return a value
- **Expressions** evaluate to a resultant value

```rust
THIS WONT WORK!
YOU CANT DO x = y = 6 IN RUST.
fn main() {
	// let y = 6 doesn't return anything so we can't bind it to x
  let x = (let y = 6);
}
```

Calling a function is an expression. Calling a macro is an expression. A new scope block created with curly brackets is an expression, for example:

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1   // Notice the lack of ; here
    };

    println!("The value of y is: {y}");
}
```

The following expression evaluates to 4.

```rust
{
  let x = 3;
  x + 1
}
```

Expressions do **not** include ending **semicolons**.

- If you add a semicolon to the end of an expression, you turn it into a statement, and it will then not return a value

### Functions with Return Values

Functions can return values to the code that calls them. 

We don’t name return values, but we must declare their type after an arrow `->`  

```rust
fn main() {
    let x = plus_one(5);
    println!("The value of x is: {x}");
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

You can return early from a function by using the `return` keyword and specifying a value

Most functions return the last expression implicitly. 

If we added a `;` after `x + 1` → `x + 1;` in the example above, we get an error:

```rust
fn plus_one(x: i32) -> i32 {
  |    --------            ^^^ expected `i32`, found `()`
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
```

The definition of the function `plus_one` says that it will return an `i32`, but statements don’t evaluate to a value, which is expressed by `()`, the unit type. 

Therefore, nothing is returned, which contradicts the function definition and results in an error