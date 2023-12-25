# Rc<T>: Reference Counted Smart Pointer

In the majority of cases, ownership is clear: you know exactly which variable owns a given value. However, there are cases when a single value might have multiple owners

`Rc<T>`: Enables multiple ownership explicitly

- Abbreviation for *reference counting*
- keeps track of the number of references to a value to determine whether or not the value is still in use
- If there are zero references to a value, the value can be cleaned up without any references becoming invalid
- Immutable

**Example:** Graph data structures

- Multiple edges might point to the same node, and that node is conceptually owned by all of the edges that point to it
- A node shouldn’t be cleaned up unless it doesn’t have any edges pointing to it and so has no owners

We use the `Rc<T>` type when we want to allocate some data on the heap for multiple parts of our program to read and we can’t determine at compile time which part will finish using the data last

**Note:** `Rc<T>` is only for use in single-threaded scenarios.

## Using `Rc<T>` to Share Data

Recall cons list example defined it using `Box<T>`

This time, create two lists that both share ownership of a third list

![Untitled](Rc%20T%20Reference%20Counted%20Smart%20Pointer/Untitled.png)

```rust
THIS WON'T WORK!
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
    let b = Cons(3, Box::new(a)); // error: use of moved value: `a` label: value moved here
    let c = Cons(4, Box::new(a)); // error: use of moved value: `a` label: value used here after move
}
```

- The `Cons` variants own the data they hold, so when we create the `b` list, `a` is moved into `b` and `b` owns `a`
- Then, when we try to use `a` again when creating `c`, we’re not allowed to because `a` has been moved

Potential solution:

- Could change the definition of `Cons` to hold references instead, but then we would have to specify lifetime parameters
- Specifying lifetime parameters → every element in the list will live at least as long as the entire list

### `std::rc::Rc::clone()`

Instead, we’ll change our definition of `List` to use `Rc<T>` in place of `Box<T>`

- Each `Cons` variant now holds a value and an `Rc<T>` pointing to a `List`
- When we create `b`, instead of taking ownership of `a`, we’ll clone the `Rc<List>` that `a` is holding
    - Increases the number of references from one to two and letting `a` and `b` share ownership of the data in that `Rc<List>`
- We’ll also clone `a` when creating `c`
    - Increases the number of references from two to three
- Every time we call `Rc::clone`, the reference count to the data within the `Rc<List>` will increase, and the data won’t be cleaned up unless there are zero references to it

**Note:** Need to add a `use` statement to bring `Rc<T>` into scope: not in the prelude

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));    // <-- Notice clone(&a)
    let c = Cons(4, Rc::clone(&a));
}
```

Could have called `a.clone()` rather than `Rc::clone(&a)`, but Rust’s convention is to use `Rc::clone` in this case

`Rc::clone` doesn’t make a deep copy of all the data like most types’ implementations of `clone` do

- Only increments the reference count, which doesn’t take much time

## Cloning an `Rc<T>` Increases the Reference Count

**Example:** Change working example above in so we can see the reference counts changing as we create and drop references to the `Rc<List>` in `a`

`Rc::strong_count` function: Returns the reference count

```rust
fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}

// Output:
// count after creating a = 1
// count after creating b = 2
// count after creating c = 3
// count after c goes out of scope = 2
```

- `Rc<List>` in `a` has an initial reference count of 1
- Each time we call `clone`, the count goes up by 1
- When `c` goes out of scope, the count goes down by 1
- Don’t have to call a function to decrease the reference count like we have to call `Rc::clone` to increase the reference count
    - Implementation of the `Drop` trait decreases the reference count automatically when an `Rc<T>` value goes out of scope