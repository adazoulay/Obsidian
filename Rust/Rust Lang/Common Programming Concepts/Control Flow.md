# Control Flow

The ability to run some code depending on wether a condition is `true` and to loop over code while a condition is `true`

## `if` Expression

An `if` statement allows you to branch code depending on condition

```rust
fn main() {
    let number = 6;
    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

Note: the condition ********must******** be a `bool`. If it is not, we get an error:

```rust
THIS WON'T WORK!
fn main() {
    let number = 3;
    if number {
        println!("number was three");
    }
}
```

Unlike languages such as Ruby and JavaScript, Rust will not automatically try to convert non-Boolean types to a Boolean.

### Using `if` in `let` statements

Because `if` is an expression, we can use it on the right side of a `let` statement to assign the outcome to a variable.

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };
    println!("The value of number is: {number}");
}
```

The `number` variable will be bound to a value based on the outcome of the `if`
 expression.

This type of each branch in the condition ********must******** be the same.

- Valid: `let number = if condition { 5 } else { 6 };`
- Invalid: `let number = if condition { 5 } else { "six" };`
    - Type missmatch: `i32`, `str`

## Repetition with Loops

Rust has three kinds of loops: 

- `loop`
- `while`
- `for`

### Repeating code with `loop`

The `loop` keyword tells Rust to execute a block of code over and over again forever or until you explicitly tell it to stop.

```rust
fn main() {
    loop {
        println!("again!");
    }
}
```

This will cause an infinite loop since we do not explicitly `break`

**Returning Values from Loops**

You might need to pass the result of that operation out of the loop to the rest of your code. To do this, you can add the value you want returned after the `break` expression you use to stop the loop.

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };
    println!("The result is {result}"); // result = 20
}
```

 That value will be returned out of the loop so you can use it, as shown above.

**************************Nesting loops**************************

If you have loops within loops, `break`and `continue` apply to the innermost loop at that point. 

You can optionally specify a *loop label* on a loop that you can then use with `break` or `continue` to specify that those keywords apply to the labeled loop instead of the innermost loop.

- Loop labels must begin with a single quote. Here’s an example with two nested loops:

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}"); // count = 2
}
```

- The outer loop has the label `'counting_up`, and it will count up from 0 to 2.
- The inner loop without a label counts down from 10 to 9.
- The first `break` that doesn’t specify a label will exit the inner loop only.
- The `break 'counting_up;` statement will exit the outer loop.

### Conditional Loops with `while`

While the condition is `true`, the loop runs. When the condition ceases to be `true`, the program calls `break`, stopping the loop.

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");
        number -= 1;
    }

    println!("LIFTOFF!!!"); // Output: 3, 2, 1, LIFTOFF!!!
} 
```

Eliminates a lot of nesting that would be necessary if you used `loop`, `if`, `else`, and `break`.

## Looping Through a Collection with `for`

a `for` loop executes some code for each item in a collection.

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}"); 
    }
} // Output: 10, 20, 30, 40, 50
```

The safety and conciseness of `for` loops make them the most commonly used loop construct in Rust

Even in situations in which you want to run some code a certain number of times, as in the countdown example that used a `while`

### Rust `Range`

`Range` generates all numbers in sequence starting from one number and ending before or up to another number.

- `0..10` generates a range from [1-9]
- `0..=10` generates a range from [1-10]

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{number}!");
    }
    println!("LIFTOFF!!!");
} // Output: 3, 2, 1, LIFTOFF!!!
```

With the `..` syntax, you can drop the initial value to start at 0

Or you could drop the trailing number to iterate up to and including the last element

You could also drop both values to take a slice of the entire string

```rust
let s = String::from("hello");
// Drop the 0
let slice = &s[0..2];
let slice = &s[..2];

let len = s.len();
// Drop the len
let slice = &s[3..len];
let slice = &s[3..];

// Slice of the entire string
let slice = &s[0..len];
let slice = &s[..];
```

### Other Slices

```rust
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3];
assert_eq!(slice, &[2, 3]);
```

## Match Control Flow

[[The match Control Flow Construct]]

## Control Flow with If let

[[Concise Control Flow with `if let`]]