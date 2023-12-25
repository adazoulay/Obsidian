# Refutability: When a Pattern Might Fail to Match

Patterns come in two forms:

- refutable: Patterns that can fail to match for some possible value
    - `Some(x)` in the expression `if let Some(x) = a_value` because if the value in the `a_value` variable is `None` rather than `Some`, the `Some(x)` pattern will not match
- Irrefutable: Patterns that will match for any possible value passed
    - `x` in the statement `let x = 5;` because `x` matches anything and therefore cannot fail to match

 Function parameters, `let` statements, and `for` loops can only accept irrefutable patterns, because the program cannot do anything meaningful when values don’t match

**Example:** What happens when we try to use a refutable pattern where Rust requires an irrefutable pattern and vice versa

```rust
THIS WON'T WORK
let Some(x) = some_option_value; // error: refutable pattern in local binding
```

If `some_option_value` was a `None` value, it would fail to match the pattern `Some(x)`, meaning the pattern is refutable

However, the `let` statement can only accept an irrefutable pattern because there is nothing valid the code can do with a `None` value

- Because we didn’t cover (and couldn’t cover!) every valid value with the pattern `Some(x)`, Rust rightfully produces a compiler error.

If we have a refutable pattern where an irrefutable pattern is needed, we can fix it by changing the code that uses the pattern

- Instead of using `let`, we can use `if let`

If the pattern doesn’t match, the code will just skip the code in the curly brackets, giving it a way to continue validly

```rust
if let Some(x) = some_option_value {
        println!("{}", x);
    }
```

**Example:** If we give `if let` a pattern that will always match, such as `x`, as shown in below, the compiler will give a warning

```rust
if let x = 5 {
        println!("{}", x); // warning: irrefutable `if let` pattern
    };
```

- This pattern will always match, so the `if let` is useless

For this reason, match arms must use refutable patterns, except for the last arm, which should match any remaining values with an irrefutable pattern

/code