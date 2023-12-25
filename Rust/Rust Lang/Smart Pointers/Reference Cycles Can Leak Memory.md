# Reference Cycles Can Leak Memory

By using `Rc<T>` and `RefCell<T>`: it’s possible to create references where items refer to each other in a cycle

- This creates memory leaks because the reference count of each item in the cycle will never reach 0, and the values will never be dropped

## Creating a Reference Cycle

**Example:** How a reference cycle might happen and how to prevent it, starting with the definition of the `List` enum and a `tail` method

```rust
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {}
```

**Note:** Using another variation of the `List` definition from prev examples

- The second element in the `Cons` variant is now `RefCell<Rc<List>>`
    - Now instead of having the ability to modify the `i32` value as we did prev, we want to modify the `List` value a `Cons` variant is pointing to
- Also added `tail` method to make it convenient for us to access the second item if we have a `Cons` variant

This code creates a list in `a` and a list in `b` that points to the list in `a`. Then it modifies the list in `a` to point to `b`, creating a reference cycle

```rust
fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle: it will overflow the stack
    // println!("a next item = {:?}", a.tail());
}
```

- We modify `a` so it points to `b` instead of `Nil`, creating a cycle
    - We do that by using the `tail` method to get a reference to the `RefCell<Rc<List>>` in `a`, which we put in the variable `link`
- Then we use the `borrow_mut` method on the `RefCell<Rc<List>>` to change the value inside from an `Rc<List>` that holds a `Nil` value to the `Rc<List>` in `b`
    
    ![Untitled](Reference%20Cycles%20Can%20Leak%20Memory/Untitled.png)
    

How this affects ref counts?

- after we change the list in `a` to point to `b`
    - Both `a` and `b` `Rc<List>` ref count is 2
- At the end of `main`, Rust drops the variable `b`
    - Decreases the reference count of the `b` `Rc<List>` instance from 2 to 1
    - Memory that `Rc<List>` has on the heap won’t be dropped at this point, because its reference count is 1, not 0.
- Then Rust drops `a`, which decreases the reference count of the `a` `Rc<List>` instance from 2 to 1 as well
    - Instance memory can’t be dropped either: The other `Rc<List>` instance still refers to it
- The memory allocated to the list will remain uncollected forever

## Preventing Reference Cycles: Turning an `Rc<T>` into a `Weak<T>`

So far, we’ve demonstrated that calling `Rc::clone` increases the `strong_count` of an `Rc<T>` instance

- `Rc<T>` instance is only cleaned up if its `strong_count` is 0

You can also create a **weak reference** to the value within an `Rc<T>` instance by calling `Rc::downgrade` and passing a reference to the `Rc<T>`.

- Strong references are how you can share ownership of an `Rc<T>` instance
- Weak references don’t express an ownership relationship, and their count doesn’t affect when an `Rc<T>` instance is cleaned up

**Weak reference** won’t create cycle because any cycle involving some weak references will be broken once the strong reference count of values involved is 0

When you call `Rc::downgrade`, you get a smart pointer of type `Weak<T>`.

- Increases the `weak_count` by 1

Because the value that `Weak<T>` references might have been dropped, to do anything with the value that a `Weak<T>` is pointing to, you must make sure the value still exists. 

`upgrade` method on a `Weak<T>` instance, returns an `Option<Rc<T>>`

- `Some` if the `Rc<T>` value has not been dropped yet
- `None` if the `Rc<T>` value has been dropped

### Creating a Tree Data Structure: a Node with Child Nodes

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}
```

- We want a `Node` to own its children, and we want to share that ownership with variables so we can access each `Node` in the tree directly
    - We define the `Vec<T>` items to be values of type `Rc<Node>`
- We also want to modify which nodes are children of another node
    - So we have a `RefCell<T>` in `children` around the `Vec<Rc<Node>>`

Next, we’ll use our struct definition and create:

- One `Node` instance named `leaf` with the value 3 and no children
- And another instance named `branch` with the value 5 and `leaf` as one of its children

```rust
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });
}
```

- We clone the `Rc<Node>` in `leaf` and store that in `branch`, meaning the `Node` in `leaf` now has two owners: `leaf` and `branch`
- We can get from `branch` to `leaf` through `branch.children`, but there’s no way to get from `leaf` to `branch`

### Adding a Reference from a Child to Its Parent

To make the child node aware of its parent, we need to add a `parent` field to our `Node` struct definition

- What type should `parent` be?
    - can’t contain an `Rc<T>` → infinite cycle with strong count
    - if a parent node is dropped, its child nodes should be dropped as well
- Therefore we should use `Weak<T>`

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```

- Creating the `leaf` node looks similar to above with the exception of the `parent` field:
    - `leaf` starts out without a parent, so we create a new, empty `Weak<Node>` reference instance
    - When we try to get a reference to the parent of `leaf` by using the `upgrade` method, we get a `None`
- Once we have the `Node` instance in `branch`, we can modify `leaf` to give it a `Weak<Node>` reference to its parent
    - Use the `borrow_mut` method on the `RefCell<Weak<Node>>` in the `parent` field of `leaf`
    - Then we use the `Rc::downgrade` function to create a `Weak<Node>` reference to `branch` from the `Rc<Node>` in `branch`
- When we print the parent of `leaf` again, this time we’ll get a `Some` variant holding `branch`

Now `leaf` can access its parent!