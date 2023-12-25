# Variables and Mutability

By default variables are immutable

We can make them mutable my adding the `mut` keyword

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```

### Constants

Like immutable variables constants are values that are bound to a name and are not allowed to change

- Declared using the `const` keyword
- Can’t use `mut` , not only immutable by default, always immutable
- Can be decalred in any scope, including global
- Constants may be set only to a constant expression, not the result of a value that could only be computed at runtime.

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Constants are valid for the entire time a program runs, within the score in which they are declared

### Shadowing

Shadowing allows us to declare a new variable with the same name as the a previous variable

ie: 1st variable is ***********************shadowed*********************** by the second →The second variable is what compiler sees

- In effect, the second variable overshadows the first, taking any uses of the variable name to itself until either it itself is shadowed or the scope ends

We can shadow a variable by using the same variable’s name and repeating the use of the `let`
 keyword as follows:

```rust
fn main() {
    let x = 5;
    let x = x + 1;
    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {x}");
    }
    println!("The value of x is: {x}");
}
// Output: 
The value of x in the inner scope is 12
The value of x is: 6
```

- Binds x to val of 5: x = 5 → creates a new var x taking original value and adding 1: x = 6
- Then, within inner scope: x = x * 2 = 12

**How is this different than mut?**

1. We need let keyword to reassign if not mut
    1. Using `let`, we can perform a few transformations on a value but have the variable be immutable after those transformations have been completed.
2. The other difference between `mut` and shadowing is that because we’re effectively creating a new variable when we use the `let` keyword again, we can change the type of the value but reuse the same name``

For example, say our program asks a user to show how many spaces they want between some text by inputting space characters, and then we want to store that input as a number:

```rust
let spaces = "   ";
let spaces = spaces.len();
```

The first `spaces`variable is a string type and the second `spaces`variable is a number type.

If we were to use mut:

```rust
let mut spaces = "   ";
spaces = spaces.len();

error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
2 |     let mut spaces = "   ";
  |                      ----- expected due to this value
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected `&str`, found `usize`
```

We cant mutate a variable’s type!