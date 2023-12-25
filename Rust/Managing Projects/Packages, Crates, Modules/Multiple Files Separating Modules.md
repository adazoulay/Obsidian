# Multiple Files: Separating Modules

When modules get large, you might want to move their definitions to a separate file to make the code easier to navigate

Example: let’s start from the code that had multiple restaurant modules

- Extract modules into files instead of having all the modules defined in the crate root file
- In this case, the crate root file is *src/lib.rs*
    - Also works with binary crates whose crate root file is *src/main.rs*
    
    ```rust
    ORIGINAL CODE: 1 FILE
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
    
1. extract the `front_of_house` module to its own file
    1. Remove the code inside the curly brackets for the `front_of_house` module, leaving only the `mod front_of_house;` declaration
    
    **Filename:** src/lib.rs
    
    ```rust
    WON'T WORK YET
    mod front_of_house;
    
    pub use crate::front_of_house::hosting;
    
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist();
    }
    ```
    
2. Place the code that was in the curly brackets into a new file named *src/front_of_house.rs*
    1. Compiler knows to look in this file because it came across the module declaration in the crate root with the name `front_of_house`
    
    **Filename:** src/front_of_house.rs
    
    ```rust
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
    ```
    
    Note: You only need to load a file using a `mod` declaration *once* in your module tree
    
    - Once the compiler knows the file is part of the project (and knows where in the module tree the code resides because of where you’ve put the `mod`
     statement), other files in your project should refer to the loaded file’s code using a path to where it was declared
        - In other words, `mod` is *not* an “include” operation that you may have seen in other programming languages.
3. Extract the `hosting` module to its own file
    1. To start moving `hosting`, we change *src/front_of_house.rs* to contain only the declaration of the `hosting` module:
        1. The process is a bit different because `hosting` is a child module of `front_of_house`, not of the root module
        2. We’ll place the file for `hosting` in a new directory that will be named for its ancestors in the module tree, in this case *src/front_of_house/*
    
    **************Filename:************** src/front_of_house.rs
    
    ```rust
    pub mod hosting;
    ```
    
4. Then we create a *src/front_of_house* directory and a file *hosting.rs* to contain the definitions made in the `hosting` module:
    
    **Filename:** src/front_of_house/hosting.rs
    
    ```rust
    pub fn add_to_waitlist() {}
    ```
    
    Note: If we instead put *hosting.rs* in the *src* directory, the compiler would expect the *hosting.rs* code to be in a `hosting` module declared in the crate root, and not declared as a child of the `front_of_house` module. 
    
    - The compiler’s rules for which files to check for which modules’ code means the directories and files more closely match the module tree
    

**Conclusion:** We’ve moved each module’s code to a separate file, and the module tree remains the same. 

- The function calls in `eat_at_restaurant` will work without any modification, even though the definitions live in different files
- This technique lets you move modules to new files as they grow in size

## Alternate File Paths

Rust also supports an older style of file path. 

For a module named `front_of_house` declared in the crate root, the compiler will look for the module’s code in:

- *src/front_of_house.rs* (what we covered)
- *src/front_of_house/mod.rs* (older style, still supported path)

For a module named `hosting` that is a submodule of `front_of_house`, the compiler will look for the module’s code in:

- *src/front_of_house/hosting.rs* (what we covered)
- *src/front_of_house/hosting/mod.rs* (older style, still supported path)

Using both styles for the same module → error 
Using a mix of both styles for different modules in the same project is allowed but confusing

Main downside to the style that uses files named *mod.rs* is that your project can end up with many files named *mod.rs*