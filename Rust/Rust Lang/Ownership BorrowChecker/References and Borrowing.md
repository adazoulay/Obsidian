# References and Borrowing

# Refferences

To avoid passing a data by ownership, we can pass it by ********************refference********************

A **reference** **is like a pointer: An address we can follow to access the data stored at that address

- That data is owned by some other variable
- Unlike a pointer, a reference is guaranteed to point to a valid value of a particular type for the life of that reference

Here is how you would define and use a `calculate_length` function that has a reference to an object as a parameter instead of taking ownership of the value

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

Here we pass `&s1` into `calculate_length` which takes in a parameter of type `&String`

The `&` represents a **********reference********** and allows us to refer to some value without taking ownership of it

Note: The opposite of referencing by using `&` is *dereferencing*, which is accomplished with the dereference operator, `*`

![[trpl04-05.svg]]

```rust
...
let s1 = String::from("hello");
let len = calculate_length(&s1);
...
fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, it is not dropped.
```

- `&s1` syntax lets us create a reference that *refers* to the value of `s1` but does not own it
    - Because it does not own it, the value it points to will **not** be dropped when the reference stops being used.
- When functions have references `&` as parameters instead of the actual values, we won’t need to return the values in order to give back ownership, because we never had ownership.
- We call the action of creating a reference **borrowing**

We **cannot** mutate something we are borrowing

```rust
WON'T WORK
fn main() {
    let s = String::from("hello");
    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world"); //`some_string` is a `&` reference, so the data
} 																	 //it refers to cannot be borrowed as mutable
```

Just as variables are immutable by default, so are references. We’re not allowed to modify something we have a reference to.

## Mutable References

We can fix the code above to allow us to modify a borrowed value using a **mutable reference**

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

- First we change `s` to be `mut`
- Then we create a mutable reference with `&mut s` where we call the `change` function, and update the function signature to accept a mutable reference with `some_string: &mut String`
- This makes it very clear that the `change` function will mutate the value it borrows

**Only one** mutable reference is allowed to access a piece of data at any given time.

- if you have a mutable reference to a value, you can have no other references to that value

This code that attempts to create two mutable references to `s` will fail

```rust
WONT WORK!
let mut s = String::from("hello");
let r1 = &mut s;
let r2 = &mut s;
println!("{}, {}", r1, r2); // cannot borrow `s` as mutable more than once at a time
```

This error says that this code is invalid because we cannot borrow `s` as mutable more than once at a time

- The first mutable borrow is in `r1` and must last until it’s used in the `println!`
- However, between the creation of that mutable reference and its usage, we tried to createanother mutable reference in `r2` that borrows the same data as `r1`

This restriction, (preventing multiple mutable references to the same data at the same time), allows for mutation but in a very controlled fashion

**data race** occurs when all 3 of these conditions occur

- Two or more pointers access the same data at the same time.
- At least one of the pointers is being used to write to the data.
- There’s no mechanism being used to synchronize access to the data.

## Scope and Refs

We can use curly brackets `{}` to create a new scope, allowing for multiple mutable references, just **not** *simultaneous* ones:

```rust
let mut s = String::from("hello");
{
	let r1 = &mut s;
} // r1 goes out of scope here, so we can make a new reference with no problems.

	let r2 = &mut s;
```

### Mutable and Immutable References

Rust enforces a similar rule for combining mutable and immutable references. This code results in an error. Can’t write to something being read by others.

```rust
DOESN'T WORK
let mut s = String::from("hello");
let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM
println!("{}, {}, and {}", r1, r2, r3);
```

However, multiple immutable references are allowed because no one who is just reading the data has the ability to affect anyone else’s reading of the data.

**Note:** a reference’s scope starts from where it is introduced and continues through the last time that reference is used.

This works:

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
println!("{} and {}", r1, r2);
// variables r1 and r2 will not be used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
```

- Compiles because the last usage of the immutable references, the `println!`, occurs before the mutable reference is introduced:
    - These scopes don’t overlap, so this code is allowed

## Dangling References

In other languages, we can have *dangling pointers*

- a pointer that references a location in memory that may have been given to someone else—by freeing some memory while preserving a pointer to that memory

The Rust compiler guarantees that references will never be dangling references

- If you have a reference to some data, the compiler will ensure that the data will not go out of scope before the reference to the data does

Dangling ref example

```rust
WON'T WORK!
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // dangle returns a reference to a String
    let s = String::from("hello"); // s is a new String
    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```

This function's return type contains a borrowed value, but there is no value for it to be borrowed from

- Because `s` is created inside `dangle`, when the code of `dangle` is finished, `s` will be deallocated.
    - But we tried to return a reference to it. That means this reference would be pointing to an invalid `String`. Rust won’t let us do this.

**Solution**: return the `String` directly

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    return s
}
```

### **Recap**

- At any given time, you can have *either* one mutable reference *or* any number of immutable references.
- References must always be valid.