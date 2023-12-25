# RefCell<T>: Interior Mutability Pattern

**Interior mutability:** Design pattern in Rust that allows you to mutate data even when there are immutable references to that data

- Normally, this action is disallowed by the borrowing rules

To mutate data, the pattern uses `unsafe` code inside a data structure to bend Rust’s usual rules that govern mutation and borrowing

- Unsafe code indicates to the compiler that we’re checking the rules manually instead of relying on the compiler

## Enforcing Borrowing Rules at Runtime with `RefCell<T>`

Unlike `Rc<T>`, the `RefCell<T>` type represents single ownership over the data it holds

What makes `RefCell<T>` different from a type like `Box<T>`?

- Recall borrowing rules:
    - At any given time, you can have *either* (but not both) one mutable reference or any number of immutable references.
    - References must always be valid
- With references and `Box<T>`, the borrowing rules’ invariants are enforced at compile time. With `RefCell<T>`, these invariants are enforced *at runtime*.
    - If you break these rules:
        - With references: Compiler error
        - With `RefCell<T>`: Program will panic and exit

`RefCell<T>` type is useful when you’re sure your code follows the borrowing rules but the compiler is unable to understand and guarantee that

`Rc<T>` and `RefCell<T>`  only used in single-threaded scenarios. Will give you a compile-time error if you try using it in a multithreaded context

## Interior Mutability: A Mutable Borrow to an Immutable Value

A consequence of the borrowing rules is that when you have an immutable value, you can’t borrow it mutably

```rust
THIS WON'T WORK
fn main() {
    let x = 5;
    let y = &mut x; // error: cannot borrow `x` as mutable, as it is not declared as mutable
}
```

However, there are situations in which it would be useful for a value to mutate itself in its methods but appear immutable to other code

- Code outside the value’s methods would not be able to mutate the value

Using `RefCell<T>` is one way to get the ability to have interior mutability, but `RefCell<T>` doesn’t get around the borrowing rules completely:

- The borrow checker in the compiler allows this interior mutability, and the borrowing rules are checked at runtime instead
- If you violate the rules, you’ll get a `panic!` instead of a compiler error

### A Use Case for Interior Mutability: Mock Objects

Mocking: Use a type in place of another type, in order to observe particular behavior and assert it’s implemented correctly

**Example:** Create a library that tracks a value against a maximum value and sends messages based on how close to the maximum value the current value is

- Could be used to keep track of a user’s quota for the number of API calls they’re allowed to make

The library doesn’t need to know that detail. All it needs is something that implements a trait we’ll provide called `Messenger`

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &'a T, max: usize) -> LimitTracker<'a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}
```

- `Messenger` trait has one method called `send` that takes an immutable reference to `self` and the text of the message
    - This trait is the interface our mock object needs to implement
- Also want to test the behavior of the `set_value` method on the `LimitTracker`
    - Can change what we pass in for the `value` parameter, but `set_value` doesn’t return anything for us to make assertions on
    - We want to be able to say that if we create a `LimitTracker` with something that implements the `Messenger` trait and a particular value for `max`, when we pass different numbers for `value`, the messenger is told to send the appropriate messages.

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

- The `sent_messages` field is now of type `RefCell<Vec<String>>` instead of `Vec<String>`.
- In the `new` function, we create a new `RefCell<Vec<String>>` instance around the empty vector.
- For the implementation of the `send` method, the first parameter is still an immutable borrow of `self`, which matches the trait definition
- We call `borrow_mut` on the `RefCell<Vec<String>>` in `self.sent_messages` to get a mutable reference to the value inside the `RefCell<Vec<String>>`, which is the vector.
    - Then we can call `push` on the mutable reference to the vector to keep track of the messages sent during the test
- The last change we have to make is in the assertion: to see how many items are in the inner vector, we call `borrow` on the `RefCell<Vec<String>>` to get an immutable reference to the vector.

### Keeping Track of Borrows at Runtime `with RefCell<T>`

When creating immutable and mutable references, we use the `&` and `&mut` syntax, respectively

With `RefCell<T>`, we use the following 2 methods:

1. `borrow` : Returns the smart pointer type `Ref<T>`
2. `borrow_mut` : Returns the smart pointer type `RefMut<T>`
- Part of the safe API that belongs to `RefCell<T>`
- Both types implement `Deref`, so we can treat them like regular references.

`RefCell<T>` keeps track of how many `Ref<T>` and `RefMut<T>` smart pointers are currently active

- When using`borrow` and`borrow_mut`, `RefCell<T>` increases its count of how many mutable/immutable borrows are active.
- When a `Ref<T>` value goes out of scope, the count of immutable borrows goes down by one.
- Follows compile-time borrowing rules: unlimited immutable borrows or **one** mutable borrow at any point in time

### Having Multiple Owners of Mutable Data by Combining `Rc<T>` and `RefCell<T>`

A common way to use `RefCell<T>` is in combination with `Rc<T>`

- Recall that `Rc<T>` lets you have multiple owners of some data, but it only gives immutable access to that data

If you have an `Rc<T>` that holds a `RefCell<T>`, you can get a value that can have multiple owners *and* that you can mutate!

******************Example:****************** Recall the cons list example

- Used `Rc<T>` to allow multiple lists to share ownership of another list
- Because `Rc<T>` holds only immutable values, we can’t change any of the values in the list once we’ve created them
- Add in `RefCell<T>` to gain the ability to change the values in the lists

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

- Create a value that is an instance of `Rc<RefCell<i32>>` and store it in a variable named `value` so we can access it directly later
- Then we create a `List` in `a` with a `Cons` variant that holds `value`
- We need to clone `value` so both `a` and `value` have ownership of the inner `5` value
    - Rather than transferring ownership from `value` to `a` or having `a` borrow from `value`
- We wrap the list `a` in an `Rc<T>` so when we create lists `b` and `c`, they can both refer to `a`
- After we’ve created the lists in `a`, `b`, and `c`, we want to add 10 to the value in `value`
    - We do this by calling `borrow_mut` on `value`, which uses the automatic dereferencing

When we print `a`, `b`, and `c`, we can see that they all have the modified value of 15 rather than 5