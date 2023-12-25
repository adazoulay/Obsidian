# Using Box<T> to Point to Heap Data

`Box<T>`: Most straightforward smart pointer

Boxes allow you to store data on the heap rather than the stack

- What remains on the stack is the pointer to the heap data

Used often in these situations:

- When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size
- When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so
- When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type

## Using ************`Box<T>` to Store Data on the Heap**

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

- Define the variable `b` to have the value of a `Box` that points to the value `5`, which is allocated on the heap
- Can access the data in the box similar to how we would if this data were on the stack
- Just like any owned value, when a box goes out of scope, as `b` does at the end of `main`, it will be deallocated
    - The deallocation happens both for the box (stored on the stack) and the data it points to (stored on the heap)

## Enabling Recursive Types with Boxess

A value of *recursive type* can have another value of the same type as part of itself

Problem: Recursive types pose an issue because at compile time Rust needs to know how much space a type takes up

- However, the nesting of values of recursive types could theoretically continue infinitely, so Rust can’t know how much space the value needs
- Because boxes have a known size, we can enable recursive types by inserting a box in the recursive type definition

********************Example:******************** cons list
Data type commonly found in functional programming languages

### More Info on Cons List

**cons list:** Data structure that comes from Lisp. Lisp version of a linked list.

- name comes from the `cons` function (short for “construct function”) that constructs a new pair from its two argument
- Calling `cons` on a pair consisting of a value and another pair, we can construct cons lists made up of recursive pairs

```lisp
(1, (2, (3, Nil)))
```

- Each item in a cons list contains two elements: the value of the current item and the next item
    - Last item in the list contains only a value called `Nil` without a next item

******************Example:****************** In Rust

- code won’t compile yet because the `List` type doesn’t have a known size,

```rust
WON'T WORK!
enum List {
    Cons(i32, List), // error: recursive type `List` has infinite size
    Nil,
}

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```

- The error shows this type “has infinite size.
- Reason: defined `List` with a variant that is recursive: it holds another value of itself directly → Rust can’t figure out how much space it needs to store a `List` value

### Computing the Size of a Non-Recursive Type

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

To determine how much space to allocate for a `Message` value, Rust goes through each of the variants to see which variant needs the most space

- Because only one variant will be used, the most space a `Message` value will need is the space it would take to store the largest of its variants

Therefore when Rust tries to determine how much space a recursive type like the `List` enum needs, it recurs infinitely

![[trpl15-01.svg]]

### Using `Box<T>` To get a Recursive Type of Known Size

Because a `Box<T>` is a pointer, Rust always knows how much space a `Box<T>` needs

- A pointer’s size doesn’t change based on the amount of data it’s pointing to

This means we can put a `Box<T>` inside the `Cons` variant instead of another `List` value directly

- The `Box<T>` will point to the next `List` value that will be on the heap rather than inside the `Cons` variant

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

`Cons` variant needs the size of an `i32` plus the space to store the box’s pointer data

We now know that any `List` value will take up **at most** the size of an `i32` plus the size of a box’s pointer data

By using a box, we’ve broken the infinite, recursive chain, so the compiler can figure out the size it needs to store a `List` value

![Untitled](Using%20Box%20T%20to%20Point%20to%20Heap%20Data/Untitled.png)

Boxes provide only the indirection and heap allocation, no performance overhead

The `Box<T>` type is a smart pointer because it implements the `Deref` trait

- Allows `Box<T>` values to be treated like references
- When a `Box<T>` value goes out of scope, the heap data that the box is pointing to is cleaned up as well because of the `Drop` trait implementation