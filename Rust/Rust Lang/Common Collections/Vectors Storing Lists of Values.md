# Vectors: Storing Lists of Values

[[Vector  Common Methods]]

`Vec<T>`: vector

- Allow you to store more than one value in a single data structure
    - Puts all the values next to each other in memory
- Vectors can only store values of the same type

## Creating A New Vector

To create a new empty vector, we call the `Vec::new` function

```rust
let v: Vec<i32> = Vec::new();
```

Note: We added a type annotation to the declaration

- Because we’re enstantiating an empty vector, Rust doesn’t know it’s type
- Vectors are implemented using generics

### `vec!` macro

More often, `Vec<T>` is created with initial values and Rust will infer the type of value you want to store

We can use the `vec!` macro to do so more concisely

```rust
let v = vec![1, 2, 3]; // Type is i32 because that’s the default integer type
```

## Updating a Vector

`push`: Adds element to `Vec<T>`

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

- The numbers we place inside are all of type `i32` so we don’t need the `Vec<i32>`
    - Rust infers this from the data
- Use `mut` to make it mutable (as with any variable)

## Reading Elements of Vectors

Two ways to reference a value stored in a vector:

- indexing
- `get` method

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {third}");

let third: Option<&i32> = v.get(2);
match third {
    Some(third) => println!("The third element is {third}"),
    None => println!("There is no third element."),
}
```

- Vectors are indexed by number, starting at zero
- Using `&` and `[]` gives us a reference to the element at the index value
- `get`method with the index passed as an argument returns `Option<&T>` that we can use with `match`

Two ways to reference an element is so you can choose how the program behaves when you try to use an index value outside the range of existing elements

```rust
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];    // PANIC!
let does_not_exist = v.get(100); // Returns Option::None
```

### Interaction with Borrow Checker

When the program has a valid reference, the borrow checker enforces the ownership and borrowing rule

- Ensure this reference and any other references to the contents of the vector remain valid

**Recall:** Can’t have mutable and immutable references in the same scope

Compiling this code will result in this error

```rust
THIS WON'T WORK!
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0]; //error: cannot borrow `v` as mutable because it is also borrowed as immutablelabel
v.push(6);
println!("The first element is: {first}");
```

- This error is due to the way vectors work
    - Because **vectors put the values next to each other in memory**, **adding a new element** onto the end of the vector **might require allocating new memory** and **copying the old elements to the new space** If there isn’t enough room to put all the elements next to each other where the vector is currently stored
- In that case, the reference to the first element would be pointing to deallocated memory.
- The borrowing rules prevent programs from ending up in that situation.

## Iterating over the Values in a Vector

Use a `for` loop to get **immutable** references to each element in a `Vec<i32>` 

```rust
let v = vec![100, 32, 57];
    for i in &v {
        println!("{i}"); // Output: 100, 32, 57
    }
```

Use a `for` loop to get **mutable** references to each element in a `Vec<i32>` 

```rust
let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;  // Notice *i
    }
```

Use the `*` dereference operator to get to the value in `i` before we can use the `+=` operator To change the value that the mutable reference refers to

Iterating over a vector, whether immutably or mutably, is safe because of the borrow checker's rules

- If we attempted to insert or remove items in the `for` loop bodies in, we would get a compiler error
- The reference to the vector that the `for` loop holds prevents simultaneous modification of the whole vector

## Using an `enum` to Store multiple Types

******************Problem:****************** Vectors can only store values that are the same type

This can be inconvenient, there are definitely use cases for needing to store a list of items of different types

******************Solution:****************** variants of an enum are defined under the same enum type

- When we need one type to represent elements of different types, we can define and use an enum

**Example:** we want to get values from a row in a spreadsheet 

- Some of the columns in the row **contain** **integers**, some **floating-point** numbers, and some **strings**

Define an `enum` whose variants will hold the different value types, and all the enum variants will be considered the same type: that of the enum

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

Rust needs to know what types will be in the vector at compile time so it knows exactly how much memory on the heap will be needed to store each element

- Using an enum plus a `match` expression means that Rust will ensure at compile time that every possible case is handled
- (can also use a trait object if type is unknown)

## Dropping a Vector Drops its Elements

Like any other `struct`, a vector is freed when it goes out of scope

```rust
{
    let v = vec![1, 2, 3, 4];

   // do stuff with v
}  // <- v goes out of scope and is freed here
```

When the vector gets dropped, all of its contents are also dropped, meaning the integers it holds will be cleaned up