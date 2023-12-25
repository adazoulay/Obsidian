# Drop Trait: Running Code on Cleanup

`Drop`: Lets you customize what happens when a value is about to go out of scope

Can provide an implementation for the `Drop` trait on any type, and that code can be used to release resources like files or network connections

`Drop` trait is almost always used when implementing a smart pointer

- Ex: When a `Box<T>` is dropped it will deallocate the space on the heap that the box points to

Requires you to implement one method named `drop` that takes a mutable reference to `self`

**Example:** `CustomSmartPointer` struct whose only custom functionality is that it will print `Dropping CustomSmartPointer!` when the instance goes out of scope

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}

// Output:
// Custom Pointers created
// Dropping CustomSmartPointer with data `other stuff`!
// Dropping CustomSmartPointer with data `My stuff`!
```

Body of the `drop` function is where you would place any logic that you wanted to run when an instance of your type goes out of scope

- In `main`, we create two instances of `CustomSmartPointer` and then print `CustomSmartPointers created`
- At the end of `main`, our instances of `CustomSmartPointer` will go out of scope, and Rust will call the code we put in the `drop` method, printing our final message
- Variables are dropped in the reverse order of their creation, so `d` was dropped before `c`

**Note:** We didn’t need to call the `drop` method explicitly

## Dropping a Value Early with `std::mem::drop`

Sometimes, might want to clean up a value early

To do so:`std::mem::drop` 

- function provided by the standard library
- Rust doesn’t let you call the `Drop` trait’s `drop` method manually; instead you have to call the `std::mem::drop` function
- Useful when using smart pointers that manage locks
    - Might want to force the `drop` method that releases the lock so that other code in the same scope can acquire the lock

Rust doesn’t let us call `drop` explicitly because Rust would still automatically call `drop` on the value at the end of `main`

- This would cause a *double free* error because Rust would be trying to clean up the same value twice

`std::mem::drop` function is different from the `drop` method in the `Drop` trait. We call it by passing as an argument the value we want to force drop

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}

// Output: 
// CustomSmartPointer created.
// Dropping CustomSmartPointer with data `some data`!
// CustomSmartPointer dropped before the end of main.
```