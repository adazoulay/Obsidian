# Deref Trait: Treating Smart Pointers Like Regular References

Implementing the `Deref` trait allows you to customize the behavior of the *dereference operator* `*`

Implementing `Deref` in such a way that a smart pointer can be treated like a regular reference, can write code that operates on references and use that code with smart pointers too

## Following the Pointer to the Value

A regular reference is a type of pointer, and one way to think of a pointer is as an arrow to a value stored somewhere else

**Example:** create a reference to an `i32` value and then use the dereference operator to follow the reference to the value

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

- variable `x` holds an `i32` value `5`
- Set `y` equal to a reference to `x`
- We can assert that `x` is equal to `5`.
- However, if we want to make an assertion about the value in `y`, we have to use `*y` to follow the reference to the value it’s pointing to (hence *dereference*) so the compiler can compare the actual value
- Once we dereference `y`, we have access to the integer value `y` is pointing to that we can compare with `5`.

## Using `Box<T>` Like a Reference

We can rewrite the code above to use a `Box<T>` instead of a reference; the dereference operator used on the `Box<T>` works the same as above

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

- Main difference between code is that here we set `y` to be an instance of a `Box<T>` pointing to a copied value of `x` rather than a reference pointing to the value of `x`
- In the last assertion, we can use the dereference operator to follow the pointer of the `Box<T>` in the same way that we did when `y` was a reference

## Defining Our Own Smart Pointer

**Note:** Big difference between `MyBox<T>`and the real `Box<T>`: our version will not store its data on the heap

- Example focuses on `Deref`, so where the data is actually stored is less important than the pointer-like behavior

```rust
struct MyBox<T>(T); // <-- Note: Tuple struct syntax. Use "." notation to access fields

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

- `MyBox` type is a tuple struct with one element of type `T`
- `MyBox::new` function takes one parameter of type `T` and returns a `MyBox` instance that holds the value passed in

Try adding the `main` function and changing it to use the `MyBox<T>` instead of `Box<T>`

```rust
THIS WON'T WORK!
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y); // error: type `MyBox<{integer}>` cannot be dereferenced
}
```

Code won’t compile because Rust doesn’t know how to dereference `MyBox`

- `Deref` trait to enable dereferencing with the `*` operator

## Implementing `Deref` Trait: Treating a Type Like a Reference

`Deref` trait, requires us to implement method `deref` that borrows `self` and returns a reference to the inner data and define type `Target`

- Provided by the standard library

```rust
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}
```

- `type Target = T;` syntax defines an associated type for the `Deref` trait to use
    - Associated types are a slightly different way of declaring a generic parameter
- Fill in the body of the `deref` method with `&self.0` so `deref` returns a reference to the value we want to access with the `*` operator

Assert equals above now pass!

- Without the `Deref` trait, the compiler can only dereference `&` references.
- The `deref` method gives the compiler the ability to take a value of any type that implements `Deref` and call the `deref` method to get a `&` reference that it knows how to dereference

Behind the scenes, when using `*y`, rust actually runs this:

```rust
*(y.deref())
```

Reason the `deref` method returns a reference to a value, and that the plain dereference outside the parentheses in `*(y.deref())` is still necessary, is to do with the ownership system

- If the `deref` method returned the value directly instead of a reference to the value
    - Value would be moved out of `self`
- We don’t want to take ownership of the inner value inside `MyBox<T>` in this case or in most cases when using `*`

## Implicit Deref Coercions with Functions and Methods

**Deref coercion** converts a reference to a type (that implements the `Deref` trait) into a reference to another type

**Example:** deref coercion can convert `&String` to `&str` because `String` implements the `Deref` trait such that it returns `&str`

Happens automatically when we pass a reference to a particular type’s value as an argument to a function/method that doesn’t match the parameter type in the function/method definition

- Added to Rust so that programmers writing function/method calls don’t need to add as many explicit references and dereferences with `&` and `*`
- also lets us write more code that can work for either references or smart pointers

**Example:** Definition of a function that has a string slice parameter on `MyBox<T>`

```rust
fn hello(name: &str) {
    println!("Hello, {name}!");
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

- Deref coercion makes it possible to call `hello` with a reference to a value of type `MyBox<String>`
- In this example, Rust uses deref coercion twice, first to get a `&String` from the `MyBox<String>`, and then to get a `&str` from the `&String`

When the `Deref` trait is defined for the types involved, Rust will analyze the types and use `Deref::deref` as many times as necessary to get a reference to match the parameter’s type

- The number of times that `Deref::deref` needs to be inserted is resolved at compile time, so there is no runtime penalty for taking advantage of deref coercion!

## How Deref Coercion Interacts with Mutability

Use the `DerefMut` trait to override the `*` operator on mutable references

Rust does deref coercion when it finds types and trait implementations in three cases:

1. From `&T` to `&U` when `T: Deref<Target=U>`
    1. If you have a `&T`, and `T` implements `Deref` to some type `U`, you can get a `&U` transparently
2. From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
    1. If you have a `&mut T`, and `T` implements `Deref` to some type `U`, you can get a `&mut U` transparently
3. From `&mut T` to `&U` when `T: Deref<Target=U>`
    1. Converting one mutable reference to one immutable reference will never break the borrowing rules → Allowed

Cases 1, 2 are the same as each other except that the second implements mutability