# Unsafe Rust

[[Raw Pointers  Common Methods]]

Rust has a second language hidden inside it that doesn’t enforce these memory safety guarantees: it’s called *unsafe Rust* and works just like regular Rust, but gives us extra superpowers.

- Exists because, by nature, static analysis is conservative

# Unsafe Superpowers

`unsafe` keyword: switch to unsafe Rust

- Then start a new block that holds the unsafe code

You can take five actions in unsafe Rust that you can’t in safe Rust

1. Dereference a raw pointer
2. Call an unsafe function or method
3. Access or modify a mutable static variable
4. Implement an unsafe trait
5. Access fields of `union`s

`unsafe` doesn’t turn off the borrow checker or disable any other of Rust’s safety checks: if you use a reference in unsafe code, it will still be checked

The `unsafe` keyword only gives you access to these five features that are then not checked by the compiler for memory safety

- You’ll still get some degree of safety inside of an unsafe block

`unsafe` does not mean the code inside the block is necessarily dangerous or that it will definitely have memory safety problems: 

- The intent is that as the programmer, you’ll ensure the code inside an `unsafe` block will access memory in a valid way
- But by requiring these five unsafe operations to be inside blocks annotated with `unsafe` you’ll know that any errors related to memory safety must be within an `unsafe` block

**Note:** Keep `unsafe` blocks small; you’ll be thankful later when you investigate memory bugs

# Dereferencing a Raw Pointer

Unsafe Rust has **two** new types called **raw pointers** that are similar to references

As with references, raw pointers can be immutable or mutable and are written as:

- `*const T` and `*mut T`
- In the context of raw pointers, *immutable* means that the pointer can’t be directly assigned to after being dereferenced

The asterisk `*` isn’t the dereference operator; it’s part of the type name

Different from references and smart pointers, raw pointers:

- Are allowed to ignore the borrowing rules by having both immutable and mutable pointers or multiple mutable pointers to the same location
- Aren’t guaranteed to point to valid memory
- Are allowed to be null
- Don’t implement any automatic cleanup

By opting out of having Rust enforce these guarantees, you can give up guaranteed safety in exchange for greater performance or the ability to interface with another language or hardware where Rust’s guarantees don’t apply

****Example:**** Shows how to create an immutable and a mutable raw pointer from references

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
```

**Note:** We don’t include the `unsafe` keyword in this code

- We can create raw pointers in safe code; we just can’t dereference raw pointers outside an unsafe block

We’ve created raw pointers by using `as` to cast an immutable and a mutable reference into their corresponding raw pointer types

Because we created them directly from references guaranteed to be valid, we know these particular raw pointers are valid, but we can’t make that assumption about just any raw pointer

****************Example:**************** Create a raw pointer whose validity we can’t be so certain of

- Shows how to create a raw pointer to an arbitrary location in memory
    - Trying to use arbitrary memory is undefined: there might be data at that address or there might not
    - No reason to do so, but possible

```rust
let address = 0x012345usize;
let r = address as *const i32;
```

We can create raw pointers in safe code, but we can’t *dereference* raw pointers and read the data being pointed to

**Example:** We use the dereference operator `*` on a raw pointer that requires an `unsafe` block

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
		*r2 += 1;
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}

// Output:
// R1 is :5
// R2 is :5
```

- If we were to print: `r1` instead of `*r1`, we would get an address like: `R1 is :0x7ff7b57eedcc`

Creating a pointer does no harm; it’s only when we try to access the value that it points at that we might end up dealing with an invalid value

**Note:** In the example above, we create a mutable and immutable reference to rust using raw pointers. Would not work in unsafe rust

When to use raw pointers?

- When interfacing with C code
- When building up safe abstractions that the borrow checker doesn’t understand

# Calling an Unsafe Function or Method

Unsafe functions and methods look exactly like regular functions and methods, but they have an extra `unsafe` before the rest of the definition

The `unsafe` keyword in this context indicates the function has requirements we need to uphold when we call this function, because Rust can’t guarantee we’ve met these requirements

**Example:** Unsafe function named `dangerous` that doesn’t do anything in its body

```rust
unsafe fn dangerous() {}

unsafe {
    dangerous();
}
```

We must call the `dangerous` function within a separate `unsafe` block

- If we try to call `dangerous` without the `unsafe` block, we’ll get an error

Bodies of unsafe functions are effectively `unsafe` blocks, so to perform other unsafe operations within an unsafe function, we don’t need to add another `unsafe` block

### Creating a Safe Abstraction over Unsafe Code

Just because a function contains unsafe code doesn’t mean we need to mark the entire function as unsafe

- Wrapping unsafe code in a safe function is a common abstraction

**Example:** `split_at_mut` function from the standard library, which requires some unsafe code. We’ll explore how we might implement it

- This safe method is defined on mutable slices: it takes one slice and makes it two by splitting the slice at the index given as an argument

```rust
let mut v = vec![1, 2, 3, 4, 5, 6];

let r = &mut v[..];

let (a, b) = r.split_at_mut(3);

assert_eq!(a, &mut [1, 2, 3]);
assert_eq!(b, &mut [4, 5, 6]);
```

We can’t implement this function using only safe Rust

An attempt might look something like this:

```rust
THIS WON'T WORK!
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();

    assert!(mid <= len);

    (&mut values[..mid], &mut values[mid..]) // error: cannot borrow `*values` as mutable more than once at a time 
}
```

- This function first gets the total length of the slice.
- Then it asserts that the index given as a parameter is within the slice by checking whether it’s less than or equal to the length
    - The assertion means that if we pass an index that is greater than the length to split the slice at, the function will panic before it attempts to use that index
- Then we return two mutable slices in a tuple: one from the start of the original slice to the `mid` index and another from `mid` to the end of the slice

Rust’s borrow checker can’t understand that we’re borrowing different parts of the slice; it only knows that we’re borrowing from the same slice twice

- Borrowing different parts of a slice is fundamentally okay because the two slices aren’t overlapping, but Rust isn’t smart enough to know this

When we know code is okay, but Rust doesn’t, it’s time to reach for unsafe code

```rust
use std::slice;

fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    let ptr = values.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
```

- Recall that slices are a pointer to some data and the length of the slice
- We use the `len` method to get the length of a slice and the `as_mut_ptr` method to access the raw pointer of a slice
- Because we have a mutable slice to `i32` values, `as_mut_ptr` returns a raw pointer with the type `*mut i32`, stored in the variable `ptr`
- Unsafe code:
    - `slice::from_raw_parts_mut` function takes a raw pointer and a length, and it creates a slice
    - We use this function to create a slice that starts from `ptr` and is `mid` items long
    - Then we call the `add` method on `ptr` with `mid` as an argument to get a raw pointer that starts at `mid`
    - We create a slice using that pointer and the remaining number of items after `mid` as the length
- The function `slice::from_raw_parts_mut` is unsafe because it takes a raw pointer and must trust that this pointer is valid. The `add` method on raw pointers is also unsafe, because it must trust that the offset location is also a valid pointer
    - By adding the assertion that `mid` must be less than or equal to `len`we can tell that all the raw pointers used within the `unsafe` block will be valid pointers to data within the slice

**Note:** We don’t need to mark the resulting `split_at_mut` function as `unsafe`, and we can call this function from safe Rust

- We’ve created a safe abstraction to the unsafe code with an implementation of the function that uses `unsafe` code in a safe way, because it creates only valid pointers from the data this function has access to

******************Example:****************** the use of `slice::from_raw_parts_mut` below would likely crash when the slice is used``

- This code takes an arbitrary memory location and creates a slice 10,000 items long

```rust
use std::slice;
let address = 0x01234usize;
let r = address as *mut i32;
let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
```

# Using `extern` Functions to Call External Code

Sometimes, your Rust code might need to interact with code written in another language

`extern` keyword: facilitates the creation and use of a *Foreign Function Interface (FFI)*

- An FFI is a way for a programming language to define functions and enable a different (foreign) programming language to call those functions

Functions declared within `extern` blocks are always unsafe to call from Rust code

- The reason is that other languages don’t enforce Rust’s rules and guarantees, and Rust can’t check them

****************Example:**************** How to set up an integration with the `abs` function from the C standard library

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

- Within the `extern "C"` block, we list the names and signatures of external functions from another language we want to call

The `"C"` part defines which *application binary interface (ABI)* the external function uses

- the ABI defines how to call the function at the assembly level
- The `"C"` ABI is the most common and follows the C programming language’s ABI

### Calling Rust Functions from Other Languages

We can also use `extern` to create an interface that allows other languages to call Rust functions

To do so:

- Instead of creating a whole `extern` block, we add the `extern` keyword and specify the ABI to use just before the `fn` keyword for the relevant function
- We also need to add a `#[no_mangle]` annotation to tell the Rust compiler not to mangle the name of this function
    - *Mangling* is when a compiler changes the name we’ve given a function to a different name that contains more information for other parts of the compilation process to consume but is less human readable

****************Example:**************** We make the `call_from_c` function accessible from C code, after it’s compiled to a shared library and linked from C:``

```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

# Accessing or Modifying a Mutable Static Variable

Not yet talked about *global variables*, which Rust does support but can be problematic with Rust’s ownership rules

- If two threads are accessing the same mutable global variable, it can cause a data race

In Rust, global variables are called `static` variables

- Static variables can only store references with the `'static` lifetime, which means the Rust compiler can figure out the lifetime and we aren’t required to annotate it explicitly

Static variables are similar to constants

- A subtle difference between constants and immutable static variables is that values in a static variable have a fixed address in memory
    - Using the value will always access the same data
    - Constants, on the other hand, are allowed to duplicate their data whenever they’re used
- Another difference is that static variables can be mutable
    - Accessing and modifying mutable static variables is *unsafe*

Names of static variables are in `SCREAMING_SNAKE_CASE` by convention

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```

********************Example:******************** Accessing and modifying mutable static variables

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

- As with regular variables, we specify mutability using the `mut` keyword
- Any code that reads or writes from `COUNTER` must be within an `unsafe` block
- This code compiles and prints `COUNTER: 3` as we would expect because it’s single threaded
    - Having multiple threads access `COUNTER` would likely result in data races
- With mutable data that is globally accessible, it’s difficult to ensure there are no data races, which is why Rust considers mutable static variables to be unsafe
- Where possible, it’s preferable to use the concurrency techniques and thread-safe smart pointers

# Implementing an Unsafe Trait

We can use `unsafe` to implement an unsafe trait

A trait is unsafe when at least one of its methods has some invariant that the compiler can’t verify

We declare that a trait is `unsafe` by adding the `unsafe` keyword before `trait` and marking the implementation of the trait as `unsafe` too

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}

fn main() {}
```

By using `unsafe impl`, we’re promising that we’ll uphold the invariants that the compiler can’t verify

As an example, recall the `Sync` and `Send` marker traits we discussed: the compiler implements these traits automatically if our types are composed entirely of `Send` and `Sync` types

- If we implement a type that contains a type that is not `Send` or `Sync`, such as raw pointers, and we want to mark that type as `Send` or `Sync`, we must use `unsafe`

# Accessing Fields of a Union

A `union` is similar to a `struct`, but only one declared field is used in a particular instance at one time. Unions are primarily used to interface with unions in C code. Accessing union fields is unsafe because Rust can’t guarantee the type of the data currently being stored in the union instance.