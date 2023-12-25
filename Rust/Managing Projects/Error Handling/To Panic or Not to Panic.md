# To Panic or Not to Panic

how do you decide when you should call `panic!` and when you should return `Result`?

could call `panic!` for any error situation, whether there’s a possible way to recover or not

- Making the decision that a situation is unrecoverable on behalf of the calling code

Choosing to return  `Result`value → give the calling code options

- Calling code could choose to attempt to recover in a way that’s appropriate for its situation
- Could decide that an `Err` value in this case is unrecoverable, so it can call `panic!`

Therefore, returning `Result` is a good default choice when you’re defining a function that might fail

In situations such as examples, prototype code, and tests, it’s more appropriate to write code that panics instead of returning a `Result`

## Examples, Prototype Code and Tests

When writing an example to illustrate some concept, including robust error-handling code can make the example less clear.

In examples, it’s understood that a call to a method like `unwrap` that could panic is meant as a placeholder for the way you’d want your application to handle errors

- `unwrap` and `expect` methods are very handy when prototyping, before you’re ready to decide how to handle errors.

## Cases in Which You Have More Information Than the Compiler

It would also be appropriate to call `unwrap` or `expect` when you have some other logic that ensures the `Result` will have an `Ok` value,

- but the logic isn’t something the compiler understands

If you can ensure by manually inspecting the code that you’ll never have an `Err` variant: 

- Perfectly acceptable to call `unwrap`
- Even better to document the reason you think you’ll never have an `Err` variant in the `expect` text

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1"
    .parse()
    .expect("Hardcoded IP address should be valid");
```

Creating an `IpAddr` instance by parsing a hardcoded string, can’t possibly Err

- `127.0.0.1` is a valid IP address, so it’s acceptable to use `expect` here

If the IP address string came from a user rather than being hardcoded  →  possibility of failure,

- Handle the `Result` in a more robust way instead

## Guidelines for Error Handling

Code panic when it’s possible that your code could end up in a bad state

In this context, a *bad state* is when some assumption, guarantee, contract, or invariant has been broken, such as:

- When invalid values, contradictory values, or missing values are passed to your code—plus one or more of the following:
    - The bad state is something that is unexpected, as opposed to something that will likely happen occasionally, like a user entering data in the wrong format
    - Your code after this point needs to rely on not being in this bad state, rather than checking for the problem at every step
    - There’s not a good way to encode this information in the types you use

## Creating Custom Types for Validation

Rust’s type system to ensure we have a valid value one step further and look at creating a custom type for validation

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```