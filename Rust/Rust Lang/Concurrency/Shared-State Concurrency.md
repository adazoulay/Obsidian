# Shared-State Concurrency

Another method to handling concurrency would be for multiple threads to access the same shared data

Shared memory concurrency is like multiple ownership

- Multiple threads can access the same memory location at the same time

## Using Mutexes to Allow Access to Data from One Thread at a Time

**Mutex***:* abbreviation for *mutual exclusion*

- A mutex allows only one thread to access some data at any given time

To access the data in a mutex, a thread must first signal that it wants access by asking to acquire the mutex’s *lock*

- The **lock** is a data structure that is part of the mutex that keeps track of who currently has exclusive access to the data

Mutexes have a reputation for being difficult to use because you have to remember two rules:

- You must attempt to acquire the lock before using the data.
- When you’re done with the data that the mutex guards, you must unlock the data so other threads can acquire the lock.

### The API of `Mutex<T>`

**Example:** How to use a mutex, single-threaded context

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

- Create a `Mutex<T>` using the associated function `new`
- `lock` method to acquire the lock and access the data inside the mutex
    - This call will block the current thread so it can’t do any work until it’s our turn to have the lock
    - Call to `lock` would fail if another thread holding the lock panicked
- After we’ve acquired the lock, we can treat the return value, named `num` in this case, as a mutable reference to the data inside
    - The type of `m` is `Mutex<i32>`, not `i32`, so we *must* call `lock` to be able to use the `i32` value

**Note:** `Mutex<T>` is a smart pointer

- More accurately, the call to `lock` *returns* a smart pointer called `MutexGuard`, wrapped in a `LockResult` that we handled with the call to `unwrap`
- `MutexGuard` smart pointer implements `Deref` to point at our inner data
- smart pointer also has a `Drop` implementation: releases the lock automatically when a `MutexGuard` goes out of scope

After dropping the lock, we can print the mutex value and see that we were able to change the inner `i32` to 6

### Sharing `Mutex<T>` Between Multiple Threads

Try to share a value between multiple threads using `Mutex<T>`

******************Example:****************** Spin up 10 threads → each increment a counter value by 1

```rust
WON'T WORK!
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || { // error: use of moved value: `counter`
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

Error message states that the `counter` value was moved in the previous iteration of the loop

- Can’t move the ownership of lock `counter` into multiple threads

### Multiple Ownership with Multiple Threads

Use `Rc<T>` to create a reference counted value

- We’ll wrap the `Mutex<T>` in `Rc<T>`

```rust
WON'T WORK!
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {   // error: `Rc<Mutex<i32>>` cannot be sent between threads safely
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

- `Rc<Mutex<i32>>` cannot be sent between threads safely`
    - The compiler is also telling us the reason why: `the trait `Send` is not implemented for `Rc<Mutex<i32>>``
- `Rc<T>` is not safe to share across threads
    - When `Rc<T>` manages the reference count, it adds to the count for each call to `clone` and subtracts from the count when each clone is dropped
    - But it doesn’t use any concurrency primitives to make sure that changes to the count can’t be interrupted by another thread

## Atomic Reference Counting with `Arc<T>`

`Arc<T>` *is* a type like `Rc<T>` that is safe to use in concurrent situations

- *a* stands for *atomic*, meaning it’s an *atomically reference counted* type
- Atomics are an additional kind of concurrency primitive: See `[std::sync::atomic](https://doc.rust-lang.org/std/sync/atomic/index.html)` for more details

Atomics work like primitive types but are safe to share across threads

Why all primitive types aren’t atomic and why standard library types aren’t implemented to use `Arc<T>` by default?

- Thread safety comes with a performance penalty that you only want to pay when you really need to

`Arc<T>` and `Rc<T>` have the same API, so we fix our program by changing the `use` line, the call to `new`, and the call to `clone`

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

## Similarities Between `RefCell<T>` / `Rc<T>` and `Mutex<T>` / `Arc<T>`

**Note:** `counter` is immutable but we could get a mutable reference to the value inside it

- Means `Mutex<T>` provides interior mutability, as the `Cell` family does
- In the same way we used `RefCell<T>` to allow us to mutate contents inside an `Rc<T>` ↔ we use `Mutex<T>` to mutate contents inside an `Arc<T>`

**Note:** Using `Rc<T>` came with the risk of creating reference cycles, where two `Rc<T>` values refer to each other, causing memory leaks

- Similarly, `Mutex<T>` comes with the risk of creating *deadlocks*
- These occur when an operation needs to lock two resources and two threads have each acquired one of the locks, causing them to wait for each other forever
    - See `MutexGuard`