# Paths: Referring to an Item in the Module Tree

To show Rust where to find an item in a module tree, we use a path in the same way we use a path when navigating a filesystem. 
To call a function, we need to know its path.

A path can take two forms:

- An *absolute path* is the full path starting from a crate root; for code from an external crate, the absolute path begins with the crate name, and for code from the current crate, it starts with the literal `crate`.
- A *relative path* starts from the current module and uses `self`, `super`, or an identifier in the current module.

Both absolute and relative paths are followed by one or more identifiers separated by double colons `::`.

[[Modules  Control Scope and Privacy]]

Looking back at Restaurant example

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}
```

we want to call the `add_to_waitlist` function

- This is the same as asking: what’s the path of the `add_to_waitlist` function

Two ways to call the `add_to_waitlist` function from a new function `eat_at_restaurant` defined in the crate root
Paths are correct, but there’s another problem remaining

```rust
THIS WON'T WORK
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist(); error: module `hosting` is private

    // Relative path
    front_of_house::hosting::add_to_waitlist(); error: module `hosting` is private
}
```

**Absolute path:**

- The `add_to_waitlist` function is defined in the same crate as `eat_at_restaurant`
    - means we can use the `crate` keyword to start an absolute path
- include each of the successive modules until we make our way to `add_to_waitlist`

**Relative path:**

- Path starts with `front_of_house`
    - The name of the module defined at the same level of the module tree as `eat_at_restaurant`

`error` messages say that module `hosting` is private
In Rust, all items (functions, methods, structs, enums, modules, and constants) are **private to parent** **modules** **by default**

- If you want to make an item like a function or struct private, you put it in a module.

Items in a **parent** **module** **can’t** use the **private** **items** **inside child** modules, but items in **child modules can** use the items in their **ancestor modules**

- Child modules wrap and hide their implementation details, but the child modules can see the context in which they’re defined

`pub` keyword: Expose inner parts of child modules’ code to outer ancestor modules

- Makes an item public

## Exposing Paths with the `pub` keyword

Mark the `hosting` module with the `pub` keyword: Gives `eat_at_restaurant` function in the **parent** module access to the `add_to_waitlist` function in the **child** module

```rust
WON'T WORK
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist(); error: function `add_to_waitlist` is private
    // Relative path
    front_of_house::hosting::add_to_waitlist(); error: function `add_to_waitlist` is private
}
```

- `pub` keyword in front of `mod hosting` makes the module public
- *Contents* of `hosting`are still **private**
    - making the module public doesn’t make its contents public.
    - The `pub` keyword on a module only lets code in its ancestor modules refer to it, not access its inner code.

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();
    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

Now the code will compile!

`front_of_house` module is defined in the crate root.

- While `front_of_house` **isn’t public,** because the `eat_at_restaurant` function is **defined in the same module** as `front_of_house`we **can** refer to `front_of_house`  from `eat_at_restaurant`
    - That is, `eat_at_restaurant` and `front_of_house` are **siblings**
- Next is the `hosting` module marked with `pub`
- We can access the parent module of `hosting` so we can access `hosting`
- Finally, the `add_to_waitlist` function is marked with `pub` and we can access its parent module

## Best Practices for Packages with a Binary and a Library

Package can contain both a *src/main.rs* binary crate root as well as a *src/lib.rs*library crate root

- Both crates will have the package name by default

Typically, packages with this pattern of containing both a library and a binary crate will have just enough code in the binary crate to start an executable that calls code with the library crate

- lets other projects benefit from the most functionality that the package provides, because the library crate’s code can be shared

The module tree should be defined in *src/lib.rs*

- Then, any public items can be used in the binary crate by starting paths with the name of the package.

## Starting Relative Paths with `super`

`super` allows us to construct relative paths that **begin** in the **parent module**, rather than the **current module** or **the crate root**

- This is like starting a filesystem path with the `..` syntax
- Allows us to reference an item that we know is in the parent module

**Example:** models the situation in which a chef fixes an incorrect order and personally brings it out to the customer

function `fix_incorrect_order` defined in the `back_of_house` module calls the function `deliver_order` **defined** in the **parent** **module** by specifying the path to `deliver_order` starting with `super`

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```

- `fix_incorrect_order` function is in the `back_of_house` module
    - Use `super` to go to the parent module of `back_of_house`, which in this case is `crate`, the root

## Making Structs and Enums Public

### Structs

We can also use `pub` to designate structs and enums as public

- There are a few details extra to the usage of `pub`with structs and enums

If we use `pub` before a `struct` definition, we make the **struct public**, but the **struct’s fields** will still be **private**

- can make each field public or not on a case-by-case basis

**Example:** Defined a public `back_of_house::Breakfast` struct

- Public `toast` field
- Private `seasonal_fruit`field

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // Change our mind about what bread we'd like
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from("blueberries"); THESE LINES WOULD CAUSE ERROR SINCE PRIVATE
    // println!("I'd like {} toast please", meal.toast);  PRIVATE
}
```

Because the `toast` field in the `back_of_house::Breakfast` struct is **public**, in `eat_at_restaurant`

- We can write and read to the `toast` field using dot notation

We can’t use the `seasonal_fruit` field in `eat_at_restaurant` because `seasonal_fruit` is private

Note: Because `back_of_house::Breakfast` is private, the struct needs to provide a public associated function that constructs an instance of `Breakfast` 

- We’ve named it `summer` here
- If `Breakfast` didn’t have such a function, we couldn’t create an instance of `Breakfast` in `eat_at_restaurant` because we couldn’t set the value of the private `seasonal_fruit` field in `eat_at_restaurant`.

### Enums

In contrast, if we make an enum public, all of its variants are then public

- We only need the `pub` before the `enum` keyword

```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```

Because we made the `Appetizer`enum public, we can use the `Soup` and `Salad` variants in `eat_at_restaurant`

Enums aren’t very useful unless their variants are public 

- It would be annoying to have to annotate all enum variants with `pub` in every case, so the **default** for **enum** **variants** is to be **public**