# Advanced Traits

# Specifying Placeholder Types in Trait Definitions with Associated Types

*Associated types* connect a type placeholder with a trait such that the trait method definitions can use these placeholder types in their signatures

The implementor of a trait will specify the concrete type to be used instead of the placeholder type for the particular implementation

That way, we can define a trait that uses some types without needing to know exactly what those types are until the trait is implemented

****************Example:**************** `std::Iterator` trait

- The associated type is named `Item` and stands in for the type of the values the type implementing the `Iterator` trait is iterating over

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

Type `Item` is a placeholder, and the `next` method’s definition shows that it will return values of type `Option<Self::Item>`

Implementors of the `Iterator` trait will specify the concrete type for `Item`, and the `next` method will return an `Option` containing a value of that concrete type

**Note:** Might seem like a similar concept to generics

- Generics allow us to define a function without specifying what types it can handle

To examine the difference between the two concepts, we’ll look at an implementation of the `Iterator` trait on a type named `Counter` that specifies the `Item` type is `u32`

```rust
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
```

This syntax seems comparable to that of generics. So why not just define the `Iterator` trait with generics:

```rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

The difference is that when using generics, we must annotate the types in each implementation;

- Because we can also implement `Iterator<String> for Counter` or any other type, we could have multiple implementations of `Iterator` for `Counter`
- In other words, when a trait has a generic parameter, it can be implemented for a type multiple times, changing the concrete types of the generic type parameters each time.
- When we use the `next` method on `Counter`, we would have to provide type annotations to indicate which implementation of `Iterator` we want to use

With associated types, we don’t need to annotate types because we can’t implement a trait on a type multiple times

## Default Generic Type Parameters and Operator Overloading

When we use generic type parameters, we can specify a default concrete type for the generic type

- Eliminates the need for implementors of the trait to specify a concrete type if the default type works

`<PlaceholderType=ConcreteType>` Syntax: specify a default type when declaring a generic type

This technique is useful is with *operator overloading*, in which you customize the behavior of an operator (such as `+`) in particular situations

**Note:** Rust doesn’t allow you to create your own operators or overload arbitrary operators

- But you can overload the operations and corresponding traits listed in `std::ops` by implementing the traits associated with the operator

****************Example:**************** `+` operator to add two `Point` instances together.

- We do this by implementing the `Add` trait on a `Point` struct

```rust
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
```

- The `add` method adds the `x` values of two `Point` instances and the `y` values of two `Point` instances to create a new `Point`
- The `Add` trait has an associated type named `Output` that determines the type returned from the `add` method

Here is the default generic type with the Add trait

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

The new part is `Rhs=Self`: this syntax is called *default type parameters*

The `Rhs` generic type parameter (short for “right hand side”) defines the type of the `rhs` parameter in the `add` method

If we don’t specify a concrete type for `Rhs` when we implement the `Add` trait, the type of `Rhs` will default to `Self`, which will be the type we’re implementing `Add` on.

- When we implemented `Add` for `Point`, we used the default for `Rhs` because we wanted to add two `Point` instances

****************Example:**************** Implementing the `Add` trait where we want to customize the `Rhs` type rather than using the default

- Two structs, `Millimeters` and `Meters`, holding values in different units

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

- To add `Millimeters` and `Meters`, we specify `impl Add<Meters>` to set the value of the `Rhs` type parameter instead of using the default of `Self`

Default type parameters useful in two main ways:

- To extend a type without breaking existing code
- To allow customization in specific cases most users won’t need

## Fully Qualified Syntax for Disambiguation: Calling Methods with the Same Name

The following code is valid:

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
```

- When we call `fly` on an instance of `Human`, the compiler defaults to calling the method that is directly implemented on the type

To call the `fly` methods from either the `Pilot` trait or the `Wizard` trait, we need to use more explicit syntax to specify which `fly` method we mean

```rust
fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}

// Output:
// This is your captain speaking.
// Up!
// *waving arms furiously*
```

**Note:** We could also write `Human::fly(&person)`, which is equivalent to the `person.fly()`

Because the `fly` method takes a `self` parameter, if we had two *types* that both implement one *trait*, Rust could figure out which implementation of a trait to use based on the type of `self`

**associated functions** that are not methods don’t have a `self` parameter

- When there are multiple types or traits that define non-method functions with the same function name, Rust doesn't always know which type you  mean
- Unless you use *fully qualified syntax*

Generally defined as: `<Type as Trait>::function(receiver_if_method, next_arg, ...);`

- For associated functions that aren’t methods, there would not be a `receiver`: there would only be the list of other arguments

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
    // println!("A baby dog is called a {}", Animal::baby_name()); <--- This Woudn't work
		println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

- Because `Animal::baby_name` doesn’t have a `self` parameter, and there could be other types that implement the `Animal` trait, Rust can’t figure out which implementation of `Animal::baby_name` we want
- To tell Rust that we want to use the implementation of `Animal` for `Dog` as opposed to the implementation of `Animal` for some other type, we need to use fully qualified syntax
    - `<Dog as Animal>::baby_name()`

## Using Supertraits to Require One Trait’s Functionality Within Another Trait

Might write a trait definition that depends on another trait: 

- For a type to implement the first trait, you want to require that type to also implement the second trait

You would do this so that your trait definition can make use of the associated items of the second trait

- The trait your trait definition is relying on is called a *supertrait* of your trait

**Examlpe:** Make an `OutlinePrint` trait with an `outline_print` method that will print a given value formatted so that it's framed in asterisks

- given a `Point` struct that implements the standard library trait `Display` to result in `(x, y)`, when we call `outline_print` on a `Point` instance that has `1` for `x` and `3` for `y`, it should print the following:

```rust
**********
*        *
* (1, 3) *
*        *
**********
```

In the implementation of the `outline_print` method, we want to use the `Display` trait’s functionality

- Therefore, we need to specify that the `OutlinePrint` trait will work only for types that also implement `Display` and provide the functionality that `OutlinePrint` needs

We can do that in the trait definition by specifying `OutlinePrint: Display`

- This technique is similar to adding a trait bound to the trait

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

Because we’ve specified that `OutlinePrint` requires the `Display` trait, we can use the `to_string` function 

- `to_string` is automatically implemented for any type that implements `Display`

If we tried to use `to_string` without adding a colon and specifying the `Display` trait after the trait name, we’d get an error saying that no method named `to_string` was found for the type `&Self` in the current scope

```rust
WON'T WORK!
struct Point {
    x: i32,
    y: i32,
}

impl OutlinePrint for Point {}  // error: `Point` doesn't implement `std::fmt::Display`
```

To fix this, we implement `Display` on `Point` and satisfy the constraint that `OutlinePrint` requires

```rust
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

Then implementing the `OutlinePrint` trait on `Point` will compile successfully, and we can call `outline_print` on a `Point` instance to display it within an outline of asterisks

## Using the Newtype Pattern to Implement External Traits on External Types

Orphan rule:  States we’re only allowed to implement a trait on a type if either the trait or the type are local to our crate

It’s possible to get around this restriction using the *newtype pattern*, which involves creating a new type in a tuple struct

- The tuple struct will have one field and be a thin wrapper around the type we want to implement a trait for
- Then the wrapper type is local to our crate, and we can implement the trait on the wrapper

**Example:** Let’s say we want to implement `Display` on `Vec<T>`

- Orphan rule prevents us from doing directly because the `Display` trait and the `Vec<T>` type are defined outside our crate
- We can make a `Wrapper` struct that holds an instance of `Vec<T>`;
- Then we can implement `Display` on `Wrapper` and use the `Vec<T>` value

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

- The implementation of `Display` uses `self.0` to access the inner `Vec<T>`, because `Wrapper` is a tuple struct and `Vec<T>` is the item at index 0 in the tuple
- Then we can use the functionality of the `Display` type on `Wrapper`

The downside of using this technique is that `Wrapper` is a new type, so it doesn’t have the methods of the value it’s holding

- We would have to implement all the methods of `Vec<T>` directly on `Wrapper` such that the methods delegate to `self.0`, which would allow us to treat `Wrapper` exactly like a `Vec<T>`
- If we wanted the new type to have every method the inner type has, implementing the `Deref` trait on the `Wrapper` to return the inner type would be a solution