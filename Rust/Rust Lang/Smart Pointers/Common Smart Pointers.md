# Common Smart Pointers

**`Box<T>`**: Boxes allow you to store data on the heap rather than the stack. What remains on the stack is the pointer to the heap data.

- It does not have any overhead, other than storing the data on the heap.
- It implements **`Deref`** so it can be dereferenced just like a regular reference with **``**.
- When it goes out of scope, the heap data gets cleaned up.
- Common methods:
    - **`Box::new(value)`**: Creates a new box.
    - **`box`**: Dereference operator to access the data.
    - **`as_ref()`**: Converts a Box<T> into a &T.
    - **`as_mut()`**: Converts a Box<T> into a &mut T.

```rust
let b = Box::new(5);
println!("b = {}", *b);
```

**`Rc<T>`** (Reference Counting): Enables multiple ownership by tracking the number of references to a value which determines when to clean up.

- It's only for use in single-threaded scenarios.
- It's useful when you want to share data but can't determine at compile time which part of code will finish using it last.
- Common methods:
    - **`Rc::new(value)`**: Creates a new Rc instance.
    - **`Rc::clone(&rc)`**: Increases the reference (strong) count and creates a new Rc instance.
    - **`Rc::strong_count(&rc)`**: Returns the number of strong (alive) Rc pointers to this value.
    - Methods associated with **`Weak<T>`**
        - **`Rc::downgrade(&rc)`**: Creates a weak pointer to this value.
        - **`Rc::weak_count(&rc)`**: Returns the number of weak Rc pointers to this value.
        - **`Rc::upgrade`** takes a **`&Weak<T>`** as an argument and returns an **`Option<Rc<T>>`**.

```rust
use std::rc::Rc;

fn main() {
    let data = Rc::new("data".to_string());
    let a = Rc::clone(&data);
    let b = Rc::clone(&data);

    println!("data: {}", data);
    println!("a: {}", a);
    println!("b: {}", b);
}
```

**`Cell<T>`**: ********TODO********

**`RefCell<T>`**: Allows for interior mutability, a design pattern in Rust that allows you to mutate data even when there are immutable references to that data.

- Common methods:
    - **`RefCell::new(value)`**: Creates a new RefCell instance.
    - **`RefCell::borrow(&self) -> Ref<T>`**: Immutably borrows the wrapped value, returning a wrapped reference (Ref).
    - **`RefCell::borrow_mut(&self) -> RefMut<T>`**: Mutably borrows the wrapped value, returning a wrapped mutable reference (RefMut).
    - **`RefCell::try_borrow(&self) -> Result<Ref<T>, BorrowError>`**: Attempts to immutably borrow the wrapped value, returning an error if the value is currently mutably borrowed.
    - **`RefCell::try_borrow_mut(&self) -> Result<RefMut<T>, BorrowMutError>`**: Attempts to mutably borrow the wrapped value, returning an error if the value is currently borrowed.

```rust
use std::cell::RefCell;

struct Car {
    model: String,
    year: RefCell<i32>,
}

fn main() {
    let car = Car {
        model: "Tesla Model 3".to_string(),
        year: RefCell::new(2019),
    };

    *car.year.borrow_mut() += 1;

    println!("{} - {}", car.model, *car.year.borrow());
}
```

**`Arc<T>`** (Atomic Reference Counting): This type, like **`Rc<T>`**, enables multiple ownership. The 'atomic' denotes that it is safe to use concurrently.

- **`Arc<T>`** is a thread-safe version of **`Rc<T>`** that uses atomic operations for its reference counting.
- This type of smart pointer enables multiple workers to own the same data and run in parallel.
- Common methods:
    - **`Arc::new(value)`**: Creates a new Arc instance.
    - **`Arc::clone(&arc)`**: Increases the reference count and creates a new Arc instance.
    - **`Arc::strong_count(&arc)`**: Returns the number of strong (alive) Arc pointers to this value.
    - **`Arc::downgrade(&arc)`**: Creates a weak pointer to this value.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

### Combining `Rc<T>` and `RefCell<T>`: Multiple Owners of Mutable Data

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}

// Output 
// a after = Cons(RefCell { value: 15 }, Nil)
// b after = Cons(RefCell { value: 3 }, Cons(RefCell { value: 15 }, Nil))
// c after = Cons(RefCell { value: 4 }, Cons(RefCell { value: 15 }, Nil))
```