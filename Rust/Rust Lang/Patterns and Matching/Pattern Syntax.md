# Pattern Syntax

# Matching

## Matching Literals

You can match patterns against literals directly

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

# Matching Named Variables

Named variables are irrefutable patterns that match any value

- However, there is a complication when you use named variables in `match` expressions
- Because `match` starts a new scope, variables declared as part of a pattern inside the `match` expression will shadow those with the same name outside the `match` construct
    - as is the case with all variables

****************Example:****************

- We declare a variable named `x` with the value `Some(5)` and a variable `y` with the value `10`
- We then create a `match` expression on the value `x`

```rust
let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {y}"),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {y}", x);
```

- The pattern in the first match arm doesn’t match the defined value of `x`, so the code continues
- Second match arm introduces a new variable named `y` that will match any value inside a `Some` value
    - Because we’re in a new scope inside the `match` expression, this is a new `y` variable, not the `y` we declared at the beginning with the value 10
    - New `y` binding will match any value inside a `Some`, which is what we have in `x`
- So the expression for the `Some(y)` arm executes and prints `Matched, y = 5`
    - If `x` had been a `None` value instead of `Some(5)`, the patterns in the first two arms wouldn’t have matched, so the value would have matched `_`
- When the `match` expression is done, its scope ends, and so does the scope of the inner `y`. The last `println!` produces `at the end: x = Some(5), y = 10`.

## Multiple Patterns

You can match multiple patterns using the `|` syntax

- Which is the pattern *or* operator

****************Example:**************** The following code we match the value of `x` against the match arms, the first of which has an *or* option

- Meaning if the value of `x` matches either of the values in that arm

```rust
let x = 1;
match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

- This code prints `one or two`

## Matching Ranges of Values with `..=`

`..=` syntax allows us to match to an inclusive range of values

- Only works with inclusive range
- Allowed with numeric and char values

**Example:** When a pattern matches any of the values within the given range, that arm will execute

```rust
let x = 5;
match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}

let y = 'c';

match y {
    'a'..='d' => println!("early ASCII letter"),
    'e'..='z' => println!("late ASCII letter"),
    _ => println!("something else"),
}
```

- If `x` is 1, 2, 3, 4, or 5, the first arm will match
- If `y` is a, b, c, d or e, the first arm with match

# Destructuring to Break Apart Values

We can also use patterns to destructure structs, enums, and tuples to use different parts of these values

## Destructuring Structs

******************Example:****************** `Point` struct with two fields, `x` and `y`

- We can break apart using a pattern with a `let` statement

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

- Creates the variables `a` and `b` that match the values of the `x` and `y` fields of the `p` struct

Example shows that the names of the variables in the pattern don’t have to match the field names of the struct

### Shorthand: Destructuring Structs

- However, it’s common to match the variable names to the field names to make it easier to remember which variables came from which fields
- Writing `let Point { x: x, y: y } = p;` contains a lot of duplication

Rust has a shorthand for patterns that match struct fields:

- You only need to list the name of the struct field, and the variables created from the pattern will have the same names

****************Example:**************** Behaves the same way as the examble above but the variables created in the `let` pattern are `x` and `y` instead of `a` and `b`

```rust
fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

- Code creates the variables `x` and `y` that match the `x` and `y` fields of the `p` variable

### Destructuring Struct Matching

We can also destructure with literal values as part of the struct pattern rather than creating variables for all the fields

- allows us to test some of the fields for particular values while creating variables to destructure the other fields

****************Example:**************** `match` expression that separates `Point` values into three cases

```rust
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {x}"),
        Point { x: 0, y } => println!("On the y axis at {y}"),
        Point { x, y } => {
            println!("On neither axis: ({x}, {y})");
        }
    }
}
```

## Destructuring Enums

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.");
        }
        Message::Move { x, y } => {
            println!("Move in the x direction {x} and in the y direction {y}");
        }
        Message::Write(text) => {
            println!("Text message: {text}");
        }
        Message::ChangeColor(r, g, b) => {
            println!("Change the color to red {r}, green {g}, and blue {b}",)
        }
    }
}
```

- For enum variants without any data: `Message::Quit`, we can’t destructure the value any further
- For struct-like enum variants: `Message::Move`, we can use a pattern similar to the pattern we specify to match structs
    - After the variant name, we place curly brackets and then list the fields with variables so we break apart the pieces to use in the code for this arm
    - In this example we use the short hand form
- For tuple-like enum variants: `Message::Write` that holds a signle element and `Message::ChangeColor` that holds a tuple with three elements the pattern is similar to the pattern we specify to match tuples
    - The number of variables in the pattern must match the number of elements in the variant we’re matching

## Destructuring Nested Structs and Enums

Matching can work on nested items too

```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("Change color to hue {h}, saturation {s}, value {v}")
        }
        _ => (),
    }
}
```

## Destructuring Structs and Tuples

We can mix, match, and nest destructuring patterns in even more complex ways

**Example:** Shows a complicated destructure where we nest structs and tuples inside a tuple and destructure all the primitive values out

```rust
let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
```

# Ignoring Values in a Pattern

It can be useful to ignore values in a pattern

- In the last arm of a `match`, to get a catchall that doesn’t actually do anything but does account for all remaining possible values
- Ignore parts of a value

## Ignoring an Entire Value with `_`

We’ve used the underscore as a wildcard pattern that will match any value but not bind to the value

This is especially useful as the last arm in a `match` expression, but we can also use it in any pattern

****************Example:**************** `_` in function parameters

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```

- This code will completely ignore the value `3` passed as the first argument, and will print `This code only uses the y parameter: 4`
- In most cases when you no longer need a particular function parameter, you would change the signature so it doesn’t include the unused parameter
    - However, can be useful when, for example, implementing a trait where you need a certain type signature but the function body doesn’t need one of the parameters

## Ignoring Parts of a Value with a Nested `_`

We can also use `_` inside another pattern to ignore just part of a value

Useful when we want to test for only part of a value but have no use for the other parts in the corresponding code we want to run

****************Example:**************** Code responsible for managing a setting’s value

- Requirements are that the user should not be allowed to overwrite an existing customization of a setting but can unset the setting and give it a value if it is currently unset

```rust
let mut setting_value = Some(5);
    let new_setting_value = Some(10);

    match (setting_value, new_setting_value) {
        (Some(_), Some(_)) => {
            println!("Can't overwrite an existing customized value");
        }
        _ => {
            setting_value = new_setting_value;
        }
    }

    println!("setting is {:?}", setting_value);
```

This code will print `Can't overwrite an existing customized value` and then `setting is Some(5)`

- In the first match arm, we don’t need to match on or use the values inside either `Some` variant, but we do need to test for the case when `setting_value` and `new_setting_value` are the `Some` variant
- In all other cases (if either `setting_value` or `new_setting_value` are `None`) expressed by the `_` pattern in the second arm, we want to allow `new_setting_value` to become `setting_value`.

****************Example:**************** We can also use underscores in multiple places within one pattern to ignore particular values

```rust
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {first}, {third}, {fifth}")
    }
}
```

This code will print `Some numbers: 2, 8, 32`, and the values 4 and 16 will be ignored.

## Ignoring Unused Variable by Starting Its Name with `_`

If you create a variable but don’t use it anywhere, Rust will usually issue a warning because an unused variable could be a bug

Can tell Rust not to warn you about the unused variable by starting the name of the variable with an underscore

```rust
fn main() {
    let _x = 5;
    let y = 10;  // warning: unused variable: `y`
}
```

**Note:** There is a subtle difference between using only `_` and using a name that starts with an underscore

- The syntax `_x` still binds the value to the variable, whereas `_` doesn’t bind at all

```rust
THIS WON'T WORK
let s = Some(String::from("Hello!"));

if let Some(_s) = s {  // error: borrow of partially moved value: `s`
    println!("found a string");
}

println!("{:?}", s);
```

We’ll receive an error because the `s` value will still be moved into `_s`, which prevents us from using `s` again

However, using the underscore by itself doesn’t ever bind to the value

```rust
let s = Some(String::from("Hello!"));

if let Some(_) = s {
    println!("found a string");
}

println!("{:?}", s);
```

Will compile without any errors because `s` doesn’t get moved into `_`: we never bind `s` to anything; it isn’t moved.

## Ignoring Remaining Parts of a Value with `..`

With values that have many parts, we can use the `..` syntax to use specific parts and ignore the rest

- Avoids the need to list underscores for each ignored value

The `..` pattern ignores any parts of a value that we haven’t explicitly matched in the rest of the pattern

******************Example:****************** We have a `Point` struct that holds a coordinate in three-dimensional space

- In the `match` expression, we want to operate only on the `x` coordinate and ignore the values in the `y` and `z` fields

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```

- We list the `x` value and then just include the `..` pattern
    - Quicker than having to list `y: _` and `z: _`, particularly with larger structs

The syntax `..` will expand to as many values as it needs to be

******************Example:****************** How to use `..` with a tuple

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {first}, {last}");
        }
    }
}
```

In this code, the first and last value are matched with `first` and `last`

- The `..` will match and ignore everything in the middle

**Note:** However, using `..` must be unambiguous

If it is unclear which values are intended for matching and which should be ignored, Rust will give us an error

**Example:** using `..` ambiguously, so it will not compile

```rust
THIS WON'T WORK
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {   // error: `..` can only be used once per tuple pattern
            println!("Some numbers: {}", second)
        },
    }
}
```

It’s impossible for Rust to determine how many values in the tuple to ignore before matching a value with `second` and then how many further values to ignore thereafter

## Extra Conditionals with Match Guards

**Match guard:** An additional `if` condition, specified after the pattern in a `match` arm, that must also match for that arm to be chosen

- Useful for expressing more complex ideas than a pattern alone allows

The condition can use variables created in the pattern

****************Example:**************** A `match` where the first arm has the pattern `Some(x)` and also has a match guard of `if x % 2 == 0`

```rust
let num = Some(4);

match num {
    Some(x) if x % 2 == 0 => println!("The number {} is even", x),
    Some(x) => println!("The number {} is odd", x),
    None => (),
}
```

This example will print `The number 4 is even`

- When `num` is compared to the pattern in the first arm, it matches, because `Some(4)` matches `Some(x)`
- Then the match guard checks whether the remainder of dividing `x` by 2 is equal to 0  →  the first arm is selected

There is no way to express the `if x % 2 == 0` condition within a pattern, so the match guard gives us the ability to express this logic

- The downside of this additional expressiveness is that the compiler doesn't try to check for exhaustiveness when match guard expressions are involved

**Note:** We mentioned in shadowing exampe that we could use match guards to solve our pattern-shadowing problem

- (we created a new variable inside the pattern in the `match` expression instead of using the variable outside the `match` → meant we couldn’t test against the value of the outer variable)

******************Example:****************** Shows how we can use a match guard to fix this problem

```rust
fn main() {
    let x = Some(10);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {n}"),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {y}", x);
}
```

This code will now print `Matched, n = 10`

- The pattern in the second match arm doesn’t introduce a new variable `y` that would shadow the outer `y`, meaning we can use the outer `y` in the match guard
    - Instead of specifying the pattern as `Some(y)`, which would have shadowed the outer `y`, we specify `Some(n)`
    - Creates a new variable `n` that doesn’t shadow anything because there is no `n` variable outside the `match`
- The match guard `if n == y` is not a pattern and therefore doesn’t introduce new variables

### Match Guard: Using or operator `|` to specify multiple patterns

You can also use the *or* operator `|` in a match guard to specify multiple patterns

- The match guard condition will apply to all the patterns

****************Example:**************** Shows the precedence when combining a pattern that uses `|` with a match guard

- The important part of this example is that the `if y` match guard applies to `4`, `5`, *and* `6`, even though it might look like `if y` only applies to `6`

```rust
let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"),
    _ => println!("no"),
}
```

- The match condition states that the arm only matches if the value of `x` is equal to `4`, `5`, or `6` *and* if `y` is `true`

```rust
// The precedence of a match guard in relation to a pattern behaves like this:
(4 | 5 | 6) if y => ...
// Rather than this:
4 | 5 | (6 if y) => ...
```

## `@` Bindings

The *at* operator `@` lets us create a variable that holds a value at the same time as we’re testing that value for a pattern match

****Example:**** We want to test that a `Message::Hello` `id` field is within the range `3..=7`

- We also want to bind the value to the variable `id_variable` so we can use it in the code associated with the arm
- We could name this variable `id`, the same as the field, but for this example we’ll use a different name

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello {
        id: id_variable @ 3..=7,
    } => println!("Found an id in range: {}", id_variable),
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    }
    Message::Hello { id } => println!("Found some other id: {}", id),
}
```

This example will print `Found an id in range: 5`

- By specifying `id_variable @` before the range `3..=7`, we’re capturing whatever value matched the range while also testing that the value matched the range pattern
- In the second arm, where we only have a range specified in the pattern, the code associated with the arm doesn’t have a variable that contains the actual value of the `id` field
    - The `id` field’s value could have been 10, 11, or 12, but the code that goes with that pattern doesn’t know which it is
    - The pattern code isn’t able to use the value from the `id` field, because we haven’t saved the `id` value in a variable
- In the last arm, where we’ve specified a variable without a range, we do have the value available to use in the arm’s code in a variable named `id`
    - The reason is that we’ve used the struct field shorthand syntax

Using `@` lets us test a value and save it in a variable within one pattern