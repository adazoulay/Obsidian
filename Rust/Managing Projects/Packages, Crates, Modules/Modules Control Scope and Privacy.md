# Modules: Control Scope and Privacy

Reference on how modules, paths, the `use` keyword, and the `pub` keyword work in the compiler, and how most developers organize their code

- **Start from the crate root**: When compiling a crate, the compiler first looks in the crate root file for code to compile
    - Usually *src/lib.rs* for a library crate or *src/main.rs* for a binary crate
- **Declaring modules**: In the crate root file, you can declare new modules; say, you declare a “garden” module with `mod garden;`. The compiler will look for the module’s code in these places:
    - Inline, within curly brackets that replace the semicolon following `mod garden`
    - In the file *src/garden.rs*
    - In the file *src/garden/mod.rs*
- **Declaring submodules**: In any file other than the crate root, you can declare submodules. For example, you might declare `mod vegetables;` in *src/garden.rs*. The compiler will look for the submodule’s code within the directory named for the parent module in these places:
    - Inline, directly following `mod vegetables`, within curly brackets instead of the semicolon
    - In the file *src/garden/vegetables.rs*
    - In the file *src/garden/vegetables/mod.rs*
- **Paths to code in modules**: Once a module is part of your crate, you can refer to code in that module from anywhere else in that same crate, as long as the privacy rules allow, using the path to the code. For example, an `Asparagus` type in the garden vegetables module would be found at `crate::garden::vegetables::Asparagus`.
- **Private vs public**: Code within a module is private from its parent modules by default. To make a module public, declare it with `pub mod` instead of `mod`. To make items within a public module public as well, use `pub` before their declarations.
    - **Child can** see **Parent, Sibling can** see **Sibling, Parent can’t** see **********Child**********
- **The `use` keyword**: Within a scope, the `use` keyword creates shortcuts to items to reduce repetition of long paths. In any scope that can refer to `crate::garden::vegetables::Asparagus`, you can create a shortcut with `use crate::garden::vegetables::Asparagus;` and from then on you only need to write `Asparagus` to make use of that type in the scope.

Here we create a binary crate named `backyard` that illustrates these rules. The crate’s directory, also named `backyard`, contains these files and directories:

```rust
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

The crate root file in this case is *src/main.rs*, and it contains:
**Filename**: src/main.rs

```rust
use crate::garden::vegetables::Asparagus;
pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {:?}!", plant);
}
```

The `pub mod garden;` line tells the compiler to include the code it finds in *src/garden.rs*, which is:
**Filename**: src/garden.rs

```rust
pub mod vegetables;
```

Here, `pub mod vegetables;` means the code in *src/garden/vegetables.rs* is included too. That code is:

```rust
#[derive(Debug)]
pub struct Asparagus {}
```

## Grouping Related Code in Modules

*Modules* let us organize code within a crate for readability and easy reuse

- allow us to control the *privacy* of items, because code within a module is `private` by **default**

Private items are internal implementation details not available for outside use

**Example:** write a library crate that provides the functionality of a restaurant

*front of house* and others as *back of house* of Restaurant

- Front House: where customers are
    - Encompasses where the hosts seat customers, servers take orders and payment, and bartenders make drinks
- Back of house: where the chefs and cooks work in the kitchen, dishwashers clean up, and managers do administrative work

To structure our crate in this way, we can organize its functions into nested modules. 

- Create a new library named `restaurant` by running `cargo new restaurant --lib`

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

- We define a module with the `mod` keyword followed by the name of the module
    - in this case, `front_of_house`
- The body of the module then goes inside curly brackets
    - Inside modules, we can place other modules
        - In this case with the modules `hosting` and `serving`
    - Modules can also hold definitions for other items, such as structs, enums, constants, traits, and functions

By using modules, we can group related definitions together and name why they’re related.

We can navigate the code based on the groups rather than having to read through all the definition making it easier to find the definitions relevant to them

The reason *src/main.rs* and *src/lib.rs* are called crate roots:

- that the contents of either of these two files form a module named `crate`at the root of the crate’s module structure, known as the *module tree*

```rust
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

If `module A` is contained inside `module B`, we say that `module A` is the **child** of `module B` and that `module B` is the **parent** of `module A`
This relationship also holds true for ****************siblings****************