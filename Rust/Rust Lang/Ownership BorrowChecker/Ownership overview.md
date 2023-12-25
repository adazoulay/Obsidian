# Ownership overview

**Ownsership** is a set of rules that govern how Rust manages memory

# The Stack and the Heap

Both the stack and the heap are parts of memory available to your code to use at runtime, but they are structured in different ways.

### Stack

- Stores values in the order it gets them and removes the values in the opposite order
    - Last in, first out
        - Adding data is called ******push******
        - Removing data is called ******pop******
    - All data stored on the stack must have a known, fixed size. Data with an unknown size at compile time or a size that might change must be stored on the heap instead

### ********Heap********

- Less organized than stack
- When putting data on the heap, ************************allocating,************************ you request a certain amount of space
    - The memory allocator finds an empty spot in the heap that is big enough, marks it as being in use and returns a *******pointer*******
    - This process is called *allocating on the heap* and is sometimes abbreviated as just *allocating* (pushing values onto the stack is not considered allocating).
- Because the pointer to the heap is a known, fixed size, the pointer can be stored in the stack
    - To access the data, you must follow the pointer

### **Tradeoffs**

- Pushing/allocating data
    - Pushing to the stack is faster than allocating on the heap
        - This is because the allocator never has to search for a place to store new data, it just pushes it to the top of the stack
    - Allocating space on the heap requires more work because the allocator must first find a big enough space to hold the data and then perform bookkeeping to prepare for the next allocation.
- Accessing data
    - Accessign data in the heap is slower than accessing data on the stack because you have to follow a pointer to get there

**********Stack:********** When your code calls a function, the values passed into the function (including, potentially, pointers to data on the heap) and the function’s local variables get pushed onto the stack. When the function is over, those values get popped off the stack.

**********heap:********** Keeping track of what parts of code are using what data on the heap, minimizing the amount of duplicate data on the heap, and cleaning up unused data on the heap so you don’t run out of space are all problems that **ownership** addresses.

# Ownership Rules

- Each value in Rust has an *owner*.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

## Variable Scope

```rust
{    // s is not valid here, it’s not yet declared
     let s = "hello"; // s is valid from this point forward
     // do stuff with s
}    // this scope is now over, and s is no longer valid
```

Hence

- When `s` comes *into* scope, it is valid.
- It remains valid until it goes *out of* scope.

## Strings

String literal: `let s = "hello"`

- String value is hardcoded into our program.
- Convenient but immutable

### The `String` ************************type

Manages data allocated on the heap → is able to store an amount of text that is unknown to us at compile time.

```rust
let s = String::from("hello");
```

The `::` operator allows us to namespace this particular `from` function under the `String`
 type rather than using some sort of name like `string_from`.

The following kind of String can be mutated

```rust
let mut s = String::from("hello");
s.push_str(", world!"); // push_str() appends a literal to a String
println!("{}", s); // This will print `hello, world!`
```

Question: Why can `String` be mutated but literals cannot?

- The difference is in how these two types deal with memory.

### Memory and Allocation

For a string literal, we know the contents at compile time, so the text is hardcoded directly into the final executable.

- Fast and efficient
- Can’t put a blob of memory into the binary for each piece of text whose size is unknown at compile time and whose size might change while running the program.

Using `String` type, in order to support a mutable, growable piece of text, we need to allocate an amount of memory on the heap, unknown at compile time, to hold the contents.

- The memory must be requested from the memory allocator at runtime.
    - Done by Rust, `String::from` requests the memory it needs
- We need a way of returning this memory to the allocator when we’re done with our `String`
    - No garbage collector, need to alloc and free memory perfectly.
        - Pair exactly one `allocate` with exactly one `free`

In Rust, the memory is automatically returned once the variable that owns it goes out of scope.

```rust
{
   let s = String::from("hello"); // s is valid from this point forward
   // do stuff with s
}  // this scope is now over, and s is no longer valid
```

Natural point at which we can return the memory our `String` needs to the allocator

- when `s` goes out of scope.

`drop` Special function called by rust when variable goes out of scope

### Variables and Data Interacting with Move

Multiple variables can interact with the same data in different ways

```rust
// **Stack**
let x = 5;
let y = x;
```

- bind the value `5`to `x`
- then make a copy of the value in `x` and bind it to `y`
    - We now have two variables, `x` and `y`, and both equal `5`

Because **integers** are simple values with a known, fixed size, and these two `5` values are pushed onto the **stack**.

```rust
// Heap
let s1 = String::from("hello");
let s2 = s1;
```

- Since complex type, does ******not****** make a copy of heap data. The `ptr`, `len`, `capacity` is copied to `s2`
    
    ![[img.svg]]
    
- A `String`is made up of three parts
    - Pointer to the memory that holds the contents of the string
    - Length
        - How much memory, in bytes, the contents of the `String`are currently using.
    - Capacity
        - Total amount of memory, in bytes, that the `String` has received from the allocator
    
    This group of data is stored on the stack. On the right is the memory on the heap that holds the contents.
    
    ![[trpl04-02.svg]]
    

When we assign `s1` to `s2`, the `String` data is copied, meaning we copy the pointer, the length, and the capacity that are on the stack.

- We do not copy the data on the heap that the pointer refers to

**Problem**: Earlier, we said that when a variable goes out of scope, Rust automatically calls the `drop` function and cleans up the heap memory for that variable. But Figure above shows both data pointers pointing to the same location.

- When `s2`and `s1` go out of scope, they will both try to free the same memory. This is known as a *double free* error

**Solution:** To ensure memory safety, after the line `let s2 = s1;`, Rust considers `s1` as no longer valid. 

- Therefore, Rust doesn’t need to free anything when `s1` goes out of scope.

```rust
THIS WON'T WORK
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{}, world!", s1); //s1 no longer holds ref to string, s2 is owner
}
```

Think of it like a ************************shallow copy************************ in other languages

In rust, it’s called a `move` because Rust invalidates the first variable 

### Variable and Data Interacting with `Clone`

If we *do* want to **deeply copy** the **heap** data of the `String`, not just the stack data, we can use a common method called `clone`

```rust
let s1 = String::from("hello");
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2);
```

`clone` ****************is expensive but **heap** data *does* get copied

### Stack-Only Data: Copy

The following data works

```rust
let x = 5;
let y = x;
println!("x = {}, y = {}", x, y);
```

Types such as **integers** that have a **known size** at compile time are stored entirely on the **stack**, so copies of the actual values are quick to make

- There’s no difference between deep and shallow copying here, so calling `clone` wouldn’t do anything different from the usual shallow copying, and we can leave it out.

Rust has a special annotation called the `Copy` trait that we can place on types that are stored on the stack

- If a type implements the `Copy` trait, variables that use it do not move, but rather are trivially copied, making them still valid after assignment to another variable.
    - Rust won’t let us annotate a type with `Copy` if the type, or any of its parts, has implemented the `Drop` trait.

What types implement the `Copy` trait? 

- General rule: any group of simple scalar values can implement `Copy`
- Nothing that requires allocation or is some form of resource can implement `Copy`
- Can implement `Copy`:
    - All the integer types, such as `u32`.
    - The Boolean type, `bool`, with values `true` and `false`.
    - All the floating-point types, such as `f64`.
    - The character type, `char`.
    - Tuples and Arrays, if they only contain types that also implement `Copy`.
        - For example, `(i32, i32)` implements `Copy`, but `(i32, String)` does not.

## Ownership and Functions

The mechanics of passing a value to a function are similar to those when assigning a value to a variable. 

Passing a variable to a function will move or copy, just as assignment does.

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

- Using `s` after the call to `takes_ownership` would throw a compile-time error
- Since `x` is copied, you can use it after `makes_copy` without causing an error

## Return Values and Scope

Returning values can also transfer ownership

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

The ownership of a variable follows the same pattern every time: 

- Assigning a value to another variable moves it
- When a variable that includes data on the heap goes out of scope, the value will be cleaned up by `drop` unless ownership of the data has been moved to another variable.

**Problem**: taking ownership and then returning ownership with every function is tedious

- What if we want to let a function use a value but not take ownership?

**Solution**: Rust does let us return multiple values using a `tuple`

```rust
fn main() {
    let s1 = String::from("hello");
    let (s2, len) = calculate_length(s1);
    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String
    (s, length)
}
```

******************Problem:****************** This is **too much work** for a concept that should be common.

**Solution:** Rust has a feature for using a value without transferring ownership, called `**references**`