# use Keyword: Paths into Scope

Having to write out the paths to call functions can feel inconvenient and repetitive

We can create a shortcut to a path with the `use` keyword once, and then use the shorter name everywhere else in the scope

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

Adding `use` and a path in a scope is similar to creating a symbolic link in the filesystem

adding `use crate::front_of_house::hosting` in the crate root, `hosting` is now a valid name in that scope 

- just as though the `hosting` module had been defined in the crate root

Paths brought into scope with `use` also check privacy, like any other paths

Note: `use` only creates the shortcut for the particular scope in which the `use` occurs

```rust
WON'T WORK
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer { 
    pub fn eat_at_restaurant() {   // Since eat_at_restaurant in customer, we can't accese our use
        hosting::add_to_waitlist();
    }
}
```

## Crating Idiomatic `use` Paths

`use crate::front_of_house::hosting` VS `use crate::front_of_house::hosting::add_to_waitlist`

```rust
...
use crate::front_of_house::hosting::add_to_waitlist; // use Includes function def
pub fn eat_at_restaurant() {
    add_to_waitlist();
}
```

Althought accomplishes the same thing, first way is an idiomatic way to bring a function into scope with `use`

- **Specifying the parent module** when calling the function **makes it clear** that the **function isn’t locally defined** while still minimizing repetition of the full path

However, when bringing in **structs**, **enums**, and other items with `use`, **it is** **idiomatic** to **specify the full path in use.**

**Example:** Idiomatic way to bring the standard library’s `HashMap` struct into the scope of a binary crate

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

Exception: if we’re bringing **two items** with the **same name** into scope with `use` statements

- Rust doesn’t allow that

## Providing New Names with the `as` Keyword

Another solution to the problem of bringing two types of the same name into the same scope with `use`

- Specify `as` after the path: we can and a new local name
    - Alias

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {}
fn function2() -> IoResult<()> {}
```

## Re-exporting Names with `pub use`

When we bring a name into scope with the `use` keyword, the name available in the new scope is private

Re-exporting: 

- Bringing an item into scope but also making that item available for others to bring into their scope.

We can combine `pub` and `use` to enable the code that calls our code to refer to that name as if it had been defined in that code’s scope

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

- Before this change, external code would have to call the `add_to_waitlist`function by using the path
    - `restaurant::front_of_house::hosting::add_to_waitlist()`
- Now that this `pub use` has re-exported the `hosting` module from the root module, external code can now use the path
    - `restaurant::hosting::add_to_waitlist()`

Re-exporting is useful when the internal structure of your code is different from how programmers calling your code would think about the domain

Using the the restaurant metaphor, the people running the restaurant think about “front of house” and “back of house”

- Customers visiting a restaurant probably won’t think about the parts of the restaurant in those terms
- With `pub use`, we can write our code with one structure but expose a different structure

## Using External Packages

To use `rand` in our project, we added this line to *Cargo.toml*

```rust
[dependencies]
rand = "0.8.5"
```

- Adding `rand` as a dependency in *Cargo.toml* tells Cargo to download the `rand` package and any dependencies from [crates.io](https://crates.io/)
- Makes `rand`available to our project

To bring `rand` definitions into the scope of our package

- We added a `use` line starting with the name of the crate, `rand`
- Then listed the items we wanted to bring into scope

```rust
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..=100);
}
```

Note: The standard `std` library is also a crate that’s external to our package

- Because the standard library is shipped with the Rust language, we don’t need to change *Cargo.toml* to include `std`
- But we do need to refer to it with `use` to bring items from there into our package’s scope
    - `use std::collections::HashMap;`

## Using Nested Paths to Clean Up Large `use` Lists

If we’re using multiple items defined in the same crate or same module, listing each item on its own line can take up a lot of vertical space in our files

Example: Two `use` statements used the Guessing Game  bring items from `std` into scope

```rust
// --snip--
use std::cmp::Ordering;
use std::io;
// --snip--
```

Instead, we can use nested paths to bring the same items into scope in one line 

Do this by 

- Specifying the common part of the path, followed by two colons
- Then curly brackets around a list of the parts of the paths that differ

```rust
use std::cmp::Ordering;
use std::io;
---- SAME ----
use std::{cmp::Ordering, io};
```

We can use a nested path at any level in a path, which is useful when combining two `use` statements that share a subpath

Example: Two `use` statements:

- One that brings `std::io` into scope
- One that brings `std::io::Write` into scope.

```rust
use std::io;
use std::io::Write;
----SAME----
use std::io::{self, Write};
```

## The Glob Operator

Glob operator: `*` 

Path followed by `*` brings *all* public items defined in a path into scope

```rust
use std::collections::*;
```

This `use` statement brings all public items defined in `std::collections` into the current scope

Be careful when using the glob operator! 

- Glob can make it harder to tell what names are in scope and where a name used in your program was defined.