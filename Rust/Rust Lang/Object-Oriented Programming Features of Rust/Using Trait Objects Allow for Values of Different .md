# Using Trait Objects: Allow for Values of Different Types

We want our library user to be able to extend the set of types that are valid in a particular situation

- Example: One limitation of vectors is that they can store elements of only one type
    - Workaround: define an enum that had variants to hold integers, floats, and text etc…
        - Good solution when our interchangeable items are a fixed set of types that we know when our code is compiled
    - Problem: Sometimes we want our library user to be able to extend the set of types that are valid in a particular situation

Solution: **trait object***:* A trait object points to both an instance of a type implementing our specified trait and a table used to look up trait methods on that type at runtime

**Creating** a trait object:

- First specify some sort of pointer, such as a `&` reference or a `Box<T>` smart pointer
- Then use the `dyn` keyword, and then specifying the relevant trait

```rust
pub trait Draw {
    fn draw(&self);
}

pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}
```

new syntax: Defines a struct named `Screen` that holds a vector named `components`

- This vector is of type `Box<dyn Draw>`, which is a trait object
- It’s a stand-in for any type inside a `Box` that implements the `Draw` trait.

On the `Screen` struct, we’ll define a method named `run` that will call the `draw` method on each of its `components`, as shown below:

```rust
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

This works differently from defining a struct that uses a generic type parameter with trait bounds

- A generic type parameter can only be substituted with one concrete type at a time, whereas trait objects allow for multiple concrete types to fill in for the trait object at runtime

**Example:** Could have defined the `Screen` struct using a generic type and a trait bound

```rust
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
where
    T: Draw,
{
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

This restricts us to a `Screen` instance that has a list of components all of type `Button` or all of type `TextField`

If you’ll only ever have homogeneous collections, using generics and trait bounds is preferable

- Because the definitions will be monomorphized at compile time to use the concrete types.

On the other hand, with the method using trait objects, one `Screen` instance can hold a `Vec<T>` that contains a `Box<Button>` as well as a `Box<TextField>`

## Implementing the Trait

Now we’ll add some types that implement the `Draw` trait

```rust
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // code to actually draw a button
    }
}
```

`width`, `height`, and `label` fields on `Button` will differ from the fields on other components

- Example: A `TextField` type might have those same fields plus a `placeholder` field
- Each of the types we want to draw on the will implement the `Draw` trait but will use different implementation

```rust
use gui::Draw;

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
    }
}
```

Can now write their `main` function to create a `Screen` instance

Can then call the `run` method on the `Screen` instance, which will call `draw` on each of the components

```rust
use gui::{Button, Screen};

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

When we wrote the library, we didn’t know that someone might add the `SelectBox` type

- However, our `Screen` implementation was able to operate on the new type and draw it because `SelectBox` implements the `Draw` trait
    - Means it implements the `draw` method

In the implementation of `run` on `Screen` in Listing 17-5, `run` doesn’t need to know what the concrete type of each component is

- Doesn’t check whether a component is an instance of a `Button` or a `SelectBox`: just calls the `draw` method on the component

By specifying `Box<dyn Draw>` as the type of the values in the `components` vector, we’ve defined `Screen` to need values that we can call the `draw` method on

The advantage of using trait objects and Rust’s type system to write code is that we never have to check whether a value implements a particular method at runtime or worry about getting errors 

- If a value doesn’t implement a method but we call it anyway: Rust won’t compile our code if the values don’t implement the traits that the trait objects need

**Example:** Won’t work: shows what happens if we try to create a `Screen` with a `String` as a component

```rust
WON'T WORK!
use gui::Screen;

fn main() {
    let screen = Screen {
        components: vec![Box::new(String::from("Hi"))],  // error: the trait bound `String: Draw` is not satisfied
    };

    screen.run();
}
```

- This error lets us know that either we’re passing something to `Screen` we didn’t mean to pass and so should pass a different type or we should implement `Draw` on `String` so that `Screen` is able to call `draw` on it

## Trait Objects Perform Dynamic Dispatch