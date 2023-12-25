# Method Syntax

********************Methods******************** are similar to functions

- Declared with the `fn` keyword and name
- Have parameters and return a value
- Contain some code that’s run when the method is called from somewhere else

They are different because

- Methods are defined within the context of a struct, enum or trait object
- their first parameter is always `self`, which represents the instance of the struct the method is being called on.

## Defining Methods

We can change the area function that takes a `Rectangle` instance as a parameter and instead make an `area` method defined on the `Rectangle` struct

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

- `impl` (implementation) block to define the function within the context of `Rectangle`
    - Everything within this `impl` block will be associated with the `Rectangle` type
- `&self` short for `self: &Self`
    - Within an `impl` block, the type `Self` is an alias for the type that the `impl` block is for.
        - Methods must have a parameter named `self` of type `Self` for their first parameter
        - Rust lets you abbreviate this with only the name `self` in the first parameter spot.
    - Note that we still need to use the `&` in front of the `self` shorthand to indicate that this method borrows the `Self` instance
    - `&mut self` If we wanted to mutate the instance that we’ve called the method on, we’d use

### Getting Params vs Calling Methods

```rust
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() { //method
        println!("The rectangle has a nonzero width; it is {}", rect1.width); // param
    }
}
```

- `rect1.width`
    - returns the value of the width filed
- `rect1.width()`
    - returns the output of the `width(&self)` method call

### Where’s the (C/C++) → Operator?

In C and C++, two different operators are used for calling methods

- `.` if you’re calling a method on the object directly
- `->` if you’re calling the method on a pointer to the object and need to dereference the pointer first

In other words, if `object` is a pointer, `object->something()` is similar to `(*object).something()`

Rust doesn’t have an equivalent to the `->` operator; instead, Rust has a feature called ***automatic referencing and dereferencing***

Here’s how it works: when you call a method with `object.something()`, Rust automatically adds in `&`, `&mut`, or `*` so `object` matches the signature of the method. 

In other words, the following are the same:

```rust
p1.distance(&p2);
(&p1).distance(&p2);
```

### Methods with More Parameters

`can_hold`: we want an instance of `Rectangle` to take another instance of `Rectangle` and return `true` if the second `Rectangle` can fit completely within `self` (the first `Rectangle`); otherwise, it should return `false`

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

### Associated Functions

All functions defined within an `impl` block are called *associated functions* because they’re associated with the type named after the `impl`

We can define associated functions that don’t have `self` as their first parameter (and thus are not methods) because they don’t need an instance of the type to work with.

- We’ve already used one function like this: the `String::from` function that’s defined on the `String` type

Associated functions that aren’t methods are often used for constructors that will return a new instance of the struct 

- These are often called `new`, but `new` isn’t a special name and isn’t built into the language

**Example**: we could choose to provide an associated function named `square` 

- Takes one dimension parameter and use that as both width and height,
- makes it easier to create a square `Rectangle`

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
...
let square = Rectangle::squre(70); // Note ::
...
```

- `Self` keywords in the return type and in the body of the function are aliases for the type that appears after the `impl` keyword
    - In this case is `Rectangle`
- To call this associated function, we use the `::` syntax with the struct name
- This function is namespaced by the struct: the `::` syntax is used for both associated functions and namespaces created by modules.

### Multiple `impl` Blocks

Each struct is allowed to have multiple `impl` blocks

There’s no reason to separate these methods into multiple `impl` blocks here, but this is valid syntax

Is useful in generic Types and Traits

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```