# Concise Control Flow with if let

The `if let` syntax lets you combine `if` and `let` into a less verbose way to handle values that match one pattern while ignoring the rest

Syntax:**`if let <pattern> = <variable>`**

**Example:** Consider a program that matches on an `Option<u8>` value in the `config_max` variable but only wants to execute code if the value is the `Some` variant

```rust
Reminder:
enum Option<T> {
    None,
    Some(T),
}
----------------------------
let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),
    }
```

- If the value is `Some`, we print out the value in the `Some` variant by binding the value to the variable `ma`
- We don’t want to do anything with the `None` value
- To satisfy the `match` expression, we have to add `_ => ()` after processing just one variant, which is annoying boilerplate code to add

Instead, we could use the more concise `if let` 

This code behaves the same way as above

```rust
let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    }
```

The syntax `if let` takes a pattern and an expression separated by an equal sign

- Works the same way as a `match`, where the expression is given to the `match` and the pattern is its first arm
- In this case, the pattern is `Some(max)`, and the `max` binds to the value inside the `Some`
- We can then use `max` in the body of the `if let` block in the same way we used `max` in the corresponding `match` arm.
- The code in the `if let` block isn’t run if the value doesn’t match the pattern.

 However, you lose the exhaustive checking that `match` enforces.

Choosing between `match` and `if let` depends on what you’re doing in your particular situation and whether gaining conciseness is an appropriate trade-off for losing exhaustive checking.

`if let` is syntax sugar for a `match` that runs code when the value matches **one** pattern and then ignores all other values

### Adding else case

We can include an `else` with an `if let`

- The block of code that goes with the `else` is the same as the block of code that would go with the `_` case in the `match`

```rust
let coin = Coin::Quarter(String::from("Alabama"));
...
let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!("State quarter from {:?}!", state),
        _ => count += 1,
    }

----- EQUIVALENT -----

let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```