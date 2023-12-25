# Using Threads to Run Code Simultaneously

Program’s code is run in a **process** **in most current operating systems

- Operating system will manage multiple processes at once

Within a program, you can also have independent parts that run simultaneously called **************threads**************

Having multiple threads: Improves performance, but it also adds complexity

Main problems with multiple threads:

- Race conditions, where threads are accessing data or resources in an inconsistent order
- Deadlocks, where two threads are waiting for each other, preventing both threads from continuing
- Bugs that happen only in certain situations and are hard to reproduce and fix reliably

The Rust standard library uses a *1:1* model of thread implementation

- rogram uses one operating system thread per one language thread

## Creating a New Thread with `spawn`

`thread::spawn`:  Creates a new thread

- Takes a closure containing the code we want to run in the new thread

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

**Note:** When the main thread of a Rust program completes, all spawned threads are shut down, whether or not they have finished running

The output for this program might be a little different every time:

```rust
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

`thread::sleep` forces a thread to stop its execution for a short duration, allowing a different thread to run

- The threads will probably take turns, but that’s not guaranteed: depends on how your operating system schedules the threads

Even though we told the spawned thread to print until `i` is 9, it only got to 5 before the main thread shut down

## Waiting for All Threads to Finish Using `join` Handles

Can fix the problem of the spawned thread not running or ending prematurely by saving the return value of `thread::spawn` in a variable

Return type of `thread::spawn` is `JoinHandle`

- `JoinHandle` is an owned value that, when we call the `join` method on it, will wait for its thread to finish

**Example: U**se the `JoinHandle` of the thread we first create  and call `join` to make sure the spawned thread finishes before `main` exits

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

Calling `join` on the handle blocks the thread currently running until the thread represented by the handle terminates.

*Blocking* a thread means that thread is prevented from performing work or exiting

```rust
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

The code from the spawned thread now executes in it’s entirety 

The two threads continue alternating, but the main thread waits because of the call to `handle.join()` and does not end until the spawned thread is finished

**Note:** If we moved the `handle.join()` call before the `for` loop in `main`, the main thread will wait for the spawned thread to finish and then run its `for` loop, so the output won’t be interleaved anymore

## Using `move` Closures with Threads

We'll often use the `move` keyword with closures passed to `thread::spawn` because the closure will then take ownership of the values it uses from the environment,

- Thus transferring ownership of those values from one thread to another

To use data from the main thread in the spawned thread, the spawned thread’s closure must capture the values it needs

******************Example:****************** Won’t Work: Attempt to create a vector in the main thread and use it in the spawned thread

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| { // error: closure may outlive the current function, but it borrows `v`, which is owned by the current function
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

- Rust *infers* how to capture `v`, and because `println!` only needs a reference to `v`, the closure tries to borrow `v`.

**problem**: Rust can’t tell how long the spawned thread will run, so it doesn’t know if the reference to `v` will always be valid.

To fix the compiler issue, we can use the `move` keyword

- By adding the `move` keyword before the closure, we force the closure to take ownership of the values it’s using rather than allowing Rust to infer that it should borrow the values

```rust
...
let v = vec![1, 2, 3];
let handle = thread::spawn(move || {
    println!("Here's a vector: {:?}", v);
});
handle.join().unwrap();
```