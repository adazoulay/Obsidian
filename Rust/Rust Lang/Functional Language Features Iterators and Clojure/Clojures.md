# Clojures

Anonymous functions you can save in a variable or pass as arguments to other functions

- can create the closure in one place and then call the closure elsewhere to evaluate it in a different context
- Unlike functions, closures can capture values from the scope in which they’re defined

## Capturing the Environment with Closures

**Example:** t-shirt company

- Company gives away an exclusive shirt to customer on mailing list as a promotion
- Customer can pick fave color → if selected free shirt has fave color
    - If customer doesn’t have fave color, get whatever color the companyhas the most of

```rust
#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColor {
    Red,
    Blue,
}

struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())   // <-- Clojure here
    }

    fn most_stocked(&self) -> ShirtColor {
        let mut num_red = 0;
        let mut num_blue = 0;

        for color in &self.shirts {
            match color {
                ShirtColor::Red => num_red += 1,
                ShirtColor::Blue => num_blue += 1,
            }
        }
        if num_red > num_blue {
            ShirtColor::Red
        } else {
            ShirtColor::Blue
        }
    }
}
```

Specify the closure expression `|| self.most_stocked()` as the argument to `unwrap_or_else`

- This closure that takes no parameters.
- Body of closure calls `self.most_stocked()`

**Note:** We’ve passed a closure that calls `self.most_stocked()` on the current `Inventory` instance. 

- The standard library didn’t need to know anything about the `Inventory` or `ShirtColor` types we defined, or the logic we want to use in this scenario
- The closure captures an immutable reference to the `self` `Inventory` instance and passes it with the code we specify to the `unwrap_or_else` method.
- Functions, on the other hand, are not able to capture their environment in this way.

## Clojure Type Annotation

More differences between functions and closures

- Closures **don’t** usually require you to **annotate** parameters or return **type** like `fn` functions
    - Required on functions because the types are part of an explicit interface exposed to your users
- Closures are typically short and relevant only within a narrow context rather than in any arbitrary scenario
    - Compiler can infer the types of the parameters and the return type

As with variables, we **can** **add type annotations** if we want to increase explicitness and clarity at the cost of being more verbose than is strictly necessary

```rust
let expensive_closure = |num: u32| -> u32 {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };
```

With type annotations added, the syntax of closures looks more similar to the syntax of functions

(added some spaces to line up the relevant parts)

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

All valid definitions that will produce the same behavior

1. `add_one_v1` line shows a function definition 
2. `add_one_v2` shows a fully annotated closure definition 
3. `add_one_v3` remove the type annotations from the closure definition
4. `add_one_v4` remove the brackets, which are optional because the closure body has only one expression

`add_one_v3` and `add_one_v4` lines require the closures to be evaluated to be able to compile because the types will be inferred from their usage

- similar to `let v = Vec::new();` needing either type annotations or values of some type to be inserted into the `Vec` for Rust to infer the type

## Clojure Type Inference

For **closure** definitions, the **compiler will infer** **one** concrete **type** for each of their parameters and for their return value.

****************Example:**************** Won’t Work

```rust
WON'T WORK!
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5); // error: mismatched types: expected struct `String`, found integer
```

- first time we call `example_closure` with the `String` value, the compiler infers the type of `x` and the return type of the closure to be `String`
- Types are then locked into the closure in `example_closure`, and we get a type error when we next try to use a different type with the same closure

## Capturing Refences or Moving Ownership

Closures can capture values from their environment in three ways: (directly map to the three ways a function can take a parameter)

1. Borrowing immutably
2. Borrowing mutably 
3. Taking ownership

Closure will decide which of these to use based on what the body of the function does with the captured values.

**Example:** Closure captures an immutable reference to the vector named `list` because it only needs an immutable reference to print the value

```rust
let list = vec![1, 2, 3];
println!("Before defining closure: {:?}", list);

let only_borrows = || println!("From closure: {:?}", list);

println!("Before calling closure: {:?}", list);
only_borrows();
println!("After calling closure: {:?}", list);
```

Can have multiple immutable references to `list` at the same time `list` is still accessible

- Before the closure definition, after the closure definition
- Before the closure is called, and after the closure is called.

************Note:************ Variable can bind to a closure definition → can later call the closure by using the variable name and parentheses as if the variable name were a function name

**Example: C**hange the closure body so that it adds an element to the `list` vector

- The closure now captures a mutable reference

```rust
let mut list = vec![1, 2, 3];
println!("Before defining closure: {:?}", list);

let mut borrows_mutably = || list.push(7);
// Adding println("{list}") would cause error! 
borrows_mutably();
println!("After calling closure: {:?}", list);
```

**Note:** that there’s no longer a `println!` between the definition and the call of the `borrows_mutably` closure 

- When `borrows_mutably` is defined, it captures a mutable reference to `list`
- We don’t use the closure again after the closure is called, so the mutable borrow ends
- Between the closure definition and the closure call, an immutable borrow to print isn’t allowed because no other borrows are allowed when there’s a mutable borrow.

### `move` Keyword

Use the `move` keyword before the parameter list to force the closure to take ownership of the values it uses in the environment even though the body of the closure doesn’t strictly need ownership

- Useful when passing a closure to a new thread to move the data so that it’s owned by new thread

```rust
use std::thread;
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    thread::spawn(move || println!("From thread: {:?}", list))
        .join()
        .unwrap();
}
```

- Spawn a new thread, giving the thread a closure to run as an argument → Closure body prints out the list
- New thread might finish before the rest of the main thread finishes, or the main thread might finish first
    - If main thread maintained ownership of `list` but ended before the new thread did and dropped `list`, the immutable reference in the thread would be invalid
    - Therefore, the compiler requires that `list` be moved into the closure given to the new thread so the reference will be valid

## Moving Captured Values out of Closures and the `Fn` Traits

- Once a closure has captured a reference or captured ownership of a value from the environment where the closure is defined → (thus affecting what is moved *into* the closure),
- The code in the body of the closure defines what happens to the references or values when the closure is evaluated later → (thus affecting what, is moved *out of* the closure).

Body can do any of the following: move a captured value out of the closure, mutate the captured value, neither move nor mutate the value, or capture nothing from the environment to begin with.

- Affects which traits the closure implements, and traits are how functions and structs can specify what kinds of closures they can use

Closures will automatically implement one, two, or all three of these `Fn` traits, in an additive fashion, depending on how the closure’s body handles the values:

### `FnOnce`, `FnMut` and `Fn`

1. `FnOnce` applies to closures that can be called once
    - All closures implement at least this trait, because all closures can be called
    - A closure that moves captured values out of its body will only implement `FnOnce` and none of the other `Fn` traits, because it can only be called once
2. `FnMut` applies to closures that don’t move captured values out of their body, but that might mutate the captured values
    - These closures can be called more than once but not more than once at the same time
3. `Fn` applies to closures that don’t move captured values out of their body and that don’t mutate captured values, as well as closures that capture nothing from their environment.
    - These closures can be called more than once without mutating their environment, which is important in cases such as calling a closure multiple times concurrently.

**Example:** Definition of `unwrap_or_else`

```rust
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

- `T` is the generic type representing the type of the value in the `Some` and return type of the `unwrap_or_else` function
- `F` is the generic type of the parameter named `f`, which is the closure we provide when calling `unwrap_or_else`
    - Trait bound specified on the generic type `F` is `FnOnce() -> T`, which means `F` must be able to be called once, take no arguments, and return a `T`

Using `FnOnce` in the trait bound expresses the constraint that `unwrap_or_else` is only going to call `f` at most one time. In this case:

- If the `Option` is `Some` → `f` won’t be called
- If the `Option` is `None` → `f` will be called once

Because all closures implement `FnOnce`, `unwrap_or_else` accepts the most different kinds of closures and is as flexible as it can be.

**Note:** Functions can implement all three of the `Fn` traits too. If what we want to do doesn’t require capturing a value from the environment, we can use the name of a function rather than a closure where we need something that implements one of the `Fn` traits. 

- For example, on an `Option<Vec<T>>` value, we could call `unwrap_or_else(Vec::new)` to get a new, empty vector if the value is `None`.

******************Example:****************** standard library method `sort_by_key` defined on slices

- Why `sort_by_key` uses `FnMut` instead of `FnOnce` for the trait bound.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    list.sort_by_key(|r| r.width);
    println!("{:#?}", list);
}
```

The reason `sort_by_key` is defined to take an `FnMut` closure is that it calls the closure multiple times: once for each item in the slice

The closure `|r| r.width` doesn’t capture, mutate, or move out anything from its environment, so it meets the trait bound requirements

```rust
THIS WON'T WORK
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];
    let mut sort_operations = vec![];
    let value = String::from("by key called"); 
    list.sort_by_key(|r| {          // <--  error: cannot move out of `value`, a captured variable in an `FnMut` closure label: captured outer variable
        sort_operations.push(value);
        r.width
    });
    println!("{:#?}", list);
}
```

This closure can be called once; trying to call it a second time wouldn’t work because `value` would no longer be in the environment to be pushed into `sort_operations` again! Therefore, this closure only implements `FnOnce`

When we try to compile this code, we get this error that `value` can’t be moved out of the closure because the closure must implement `FnMut`

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut num_sort_operations = 0;
    list.sort_by_key(|r| {
        num_sort_operations += 1;
        r.width
    });
    println!("{:#?}, sorted in {num_sort_operations} operations", list);
}
```

The closure above works with `sort_by_key` because it is only capturing a mutable reference to the `num_sort_operations` counter and can therefore be called more than once: