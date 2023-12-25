# The match Control Flow Construct

`match` allows you to compare a value against a series of patterns and then execute code based on which pattern matches.

- Patterns can be made up of literal values, variable names, wildcards, and many other things

Power of `match` comes from the expressiveness of the patterns and the fact that the compiler confirms that all possible cases are handled.

**Example:** Coins

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!"); // Using {} we can execute multiple lines of code
            1
        }
        Coin::Nickel => 5,// Don’t typically use {} if the match arm code is short
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
OR
impl Coin {
    fn value_in_cents(&self) -> u8 {
        return match self {
            Coin::Penny => 1,
            Coin::Nickel => 5,
            Coin::Dime => 10,
            Coin::Quarter => 25,
        };
    }
 }
```

- `match` keyword followed by an expression, in this case is the value `coin: Coin`
    - Difference with `if`: the condition needs to evaluate to a Boolean value, but here it can be any type.
        - The type of `coin` in this example is the `Coin` enum that we defined on the first line
- `match` arms. An arm has two parts: a pattern and some code.
    - The first arm here has a pattern that is the value `Coin::Penny`
    - The `=>`operator that separates the pattern and the code to run.
    - Each arm is separated from the next with a comma.
- Not covering all possible cases throws an error

## Patterns That Bind to Values

Match arms can bind to the parts of the values that match the pattern

- This is how we can extract values out of enum variants

Ex: Each US state has a unique quarter

```rust
#[derive(Debug)] 
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

- In the match expression for this code, we add a variable called `state` to the pattern that matches values of the variant `Coin::Quarter`
- When a `Coin::Quarter` matches, the `state` variable will bind to the value of that quarter’s state.

Then we can use `state` in the code for that arm, like so:

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

If we were to call `value_in_cents(Coin::Quarter(UsState::Alaska))`

- `coin` would be `Coin::Quarter(UsState::Alaska)`

## Matching with `Option<T>`

If we want to get the inner `T` value out of the `Some` case when using `Option<T>`
we can also handle `Option<T>` using `match`

**Example**: We want to write a function that takes an `Option<i32>` and, if there’s a value inside, adds 1 to that value. If there isn’t a value inside, return the `None` value and not attempt to perform any operations.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

### Matches Are Exhaustive

`match`arms’ patterns must cover all possibilities, they are exhaustive

```rust
THIS WON'T WORK
fn plus_one(x: Option<i32>) -> Option<i32> {
        match x { // pattern `None` not covered
            Some(i) => Some(i + 1),
        }
    }
```

## Catch-all Patterns and the _ Placeholder

Using enums, we can also take special actions for a few particular values, but for all other values take one default action

- Similar to `swtich` `default`

**Example:** Dice game

- Roll a 3: your player doesn’t move, but instead gets a new fancy hat
- Roll a 7: your player loses a fancy hat
- All other values, your player moves that number of spaces on the game board

```rust
let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
```

- First two arms cover literal values `3` and `7`
- Last arm that covers every other possible value, in this case `i32`

### `_` Catch all

`_` is a special pattern that matches any value and does not bind to that value

- can use when we want a catch-all but don’t want to *use* the value in the catch-all pattern
- Tells Rust we aren’t going to use the value, so Rust won’t warn us about an unused variable.

To demonstrate, now if you roll anything other than a 3 or a 7, you must roll again

```rust
let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
```

Also meets the exhaustiveness requirement because we’re explicitly ignoring all other values in the last arm

Change the rules of the game one more time so that nothing else happens on your turn if you roll anything other than a 3 or a 7

```rust
let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (), // Unit value: Do Nothing
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
```