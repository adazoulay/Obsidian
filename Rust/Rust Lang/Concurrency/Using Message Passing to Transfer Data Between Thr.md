# Using Message Passing to Transfer Data Between Threads

Increasingly popular approach to ensuring safe concurrency is **message passing***:* where threads or actors communicate by sending each other messages containing data

- Do not communicate by sharing memory; instead, share memory by communicating
- To accomplish message-sending concurrency, Rust's standard library provides an implementation of **channels**

A **channel** is a general programming concept by which data is sent from one thread to another

- Can imagine a channel in programming as being like a directional channel of water, such as a stream or a river
    - If you put something like a rubber duck into a river, it will travel downstream to the end of the waterway

A channel has two halves: a transmitter and a receiver

- One part of your code calls methods on the transmitter with the data you want to send
    - Transmitter half is the upstream location where you put rubber ducks into the river
- Another part checks the receiving end for arriving messages
    - Receiver half is where the rubber duck ends up downstream
- A channel is said to be *closed* if either the transmitter or receiver half is dropped

**Example:** We’ll work up to a program that has one thread to generate values and send them down a channel, and another thread that will receive the values and print them out

- Won’t compile yet because Rust can’t tell what type of values we want to send over the channel.

```rust
THIS WON'T WORK
use std::sync::mpsc;
fn main() {
    let (tx, rx) = mpsc::channel();
}
```

- We create a new channel using the `mpsc::channel` function
    - `mpsc` stands for *multiple producer, single consumer*
    - The way Rust’s standard library implements channels means a channel can have multiple *sending* ends that produce values but only one *receiving* end that consumes those values
        - Imagine multiple streams flowing together into one big river

`mpsc::channel` function returns a tuple

- `tx`: First element of which is the sending end--the transmitter
- `rx`: Second element is the receiving end--the receiver

Now we move the transmitting end into a spawned thread and have it send one string so the spawned thread is communicating with the main thread

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

- `thread::spawn` to create a new thread and then using `move` to move `tx` into the closure so the spawned thread owns `tx`
- The spawned thread needs to own the transmitter to be able to send messages through the channel

The transmitter has a `send` method that takes the value we want to send

- `send` method returns a `Result<T, E>` type, so if the receiver has already been dropped and there’s nowhere to send a value, the send operation will return an error

The receiver has two useful methods: `recv` and `try_recv`

- `recv` blocks the main thread’s execution and wait until a value is sent down the channel. Once a value is sent, `recv` will return it in a `Result<T, E>`
    - When the transmitter closes, `recv` will return an error to signal that no more values will be coming
- `try_recv` method doesn’t block, instead returns a `Result<T, E>` immediately:
    - an `Ok` value holding a message if one is available and an `Err` value if there aren’t any messages this time.
    - Useful if this thread has other work to do while waiting for messages
        - Could write a loop that calls `try_recv` every so often, handles a message if one is available

## Channels and Ownership Transference

Ownership rules play a vital role in message sending because they help you write safe, concurrent code

**Example:** Won’t work: Experiment to show how channels and ownership work together to prevent problems

- try to use a `val` value in the spawned thread *after* we’ve sent it down the channel

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val); // error: borrow of moved value: `val`
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

- Here, we try to print `val` after we’ve sent it down the channel via `tx.send`
- Allowing this would be a bad idea: once the value has been sent to another thread, that thread could modify or drop it before we try to use the value again

## Sending Multiple Values and Seeing the Reciever Waiting

******************Example:****************** The spawned thread will now send multiple messages and pause for a second between each message

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

- This time, the spawned thread has a vector of strings that we want to send to the main thread.
- iterate over them, sending each individually, and pause between each by calling the `thread::sleep` function with a `Duration` value of 1 second
- not calling the `recv` function explicitly anymore: instead, we’re treating `rx` as an iterator
    - For each value received, we’re printing it
- When the channel is closed, iteration will end

## Creating Multiple Producers by Cloning the Transmitter

********************Remember:******************** `mpsc` is an acronym for *multiple producer, single consumer*

We can create multiple threads that all send values to the same receiver

- Done by cloning the transmitter

```rust
// --snip--
    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }

    // --snip--
```

This time, before we create the first spawned thread, we call `clone` on the transmitter

- Give us a new transmitter we can pass to the first spawned thread

We pass the original transmitter to a second spawned thread

- This gives us two threads, each sending different messages to the one receiver.