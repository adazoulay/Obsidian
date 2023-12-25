# Advanced Types

# Using the Newtype Pattern for Type Safety and Abstraction

The newtype pattern is also useful for statically enforcing that values are never confused and indicating the units of a value

Recall the `Millimeters` and `Meters` structs wrapped `u32` values in a newtype

- If we wrote a function with a parameter of type `Millimeters`, we couldn’t compile a program that accidentally tried to call that function with a value of type `Meters` or a plain `u32`

We can also use the newtype pattern to abstract away some implementation details of a type: the new type can expose a public API that is different from the API of the private inner type

Newtypes can also hide internal implementation

- example, we could provide a `People` type to wrap a `HashMap<i32, String>` that stores a person’s ID associated with their name.
- Code using `People` would only interact with the public API we provide, such as a method to add a name string to the `People` collection; that code wouldn’t need to know that we assign an `i32` ID to names internally

The newtype pattern is a lightweight way to achieve encapsulation to hide implementation details

## Creating Type Synonyms with Type Aliases

Rust provides the ability to declare a *type alias* to give an existing type another name

For this we use the `type` keyword

```rust
type Kilometers = i32;
```

The alias `Kilometers` is a *synonym* for `i32`

- Unlike the `Millimeters` and `Meters` types, `Kilometers` is not a separate, new type.
- Values that have the type `Kilometers` will be treated the same as values of type `i32`

```rust
type Kilometers = i32;

let x: i32 = 5;
let y: Kilometers = 5;

println!("x + y = {}", x + y);
```

Because `Kilometers` and `i32` are the same type, we can add values of both types and we can pass `Kilometers` values to functions that take `i32` parameters

However, using this method, we don’t get the type checking benefits that we get from the newtype pattern discussed earlier. 

- In other words, if we mix up `Kilometers` and `i32` values somewhere, the compiler will not give us an error

The main use case for type synonyms is to reduce repetition.

For example, we might have a lengthy type like this:

```rust
Box<dyn Fn() + Send + 'static>
```

Writing this lengthy type in function signatures and as type annotations all over the code can be tiresome and error prone

A type alias makes this code more manageable by reducing the repetition

```rust
let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("hi"));
fn takes_long_type(f: Box<dyn Fn() + Send + 'static>) {// --snip--}
fn returns_long_type() -> Box<dyn Fn() + Send + 'static> {// --snip--}
---- VS ----
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));
fn takes_long_type(f: Thunk) {// --snip--}
fn returns_long_type() -> Thunk {// --snip--}
```

This code is much easier to read and write

- Choosing a meaningful name for a type alias can help communicate your intent as well

Type aliases are also commonly used with the `Result<T, E>` type for reducing repetition

## The Never Type that Never Returns

Rust has a special type named `!` that’s known in type theory lingo as the *empty type* because it has no values

We prefer to call it the *never type* because it stands in the place of the return type when a function will never return

**Example:**

```rust
fn bar() -> ! {
    // --snip--
}
```

This code is read as “the function `bar` returns never.”

**diverging functions:** Functions that return

- We can’t create values of the type `!` so `bar` can never possibly return.

But what use is a type you can never create values for?

Recall this example:

```rust
let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
```

`match` arms must all return the same type. So, for example, the following code doesn’t work

```rust
THIS WON'T WORK
let guess = match guess.trim().parse() {
        Ok(_) => 5,
        Err(_) => "hello",
    };
```

The type of `guess` in this code would have to be an integer *and* a string, and Rust requires that `guess` have only one type

So what does `continue` return?

- `continue` has a `!` value
- Can also do this with `panic!` and `loop` because they have the `!` type
- That is, when Rust computes the type of `guess`, it looks at both match arms, the former with a value of `u32` and the latter with a `!` value.
- Because `!` can never have a value, Rust decides that the type of `guess` is `u32`

Formal way of describing this behavior is that expressions of type `!` can be coerced into any other type

## Dynamically Sized Types and the `Sized` Trait

Rust needs to know certain details about its types, such as how much space to allocate for a value of a particular type

This leaves one corner of its type system a little confusing at first: 

- The concept of *dynamically sized types*

Sometimes referred to as *DSTs* or *unsized types*, these types let us write code using values whose size we can know only at runtime

Let’s dig into the details of a dynamically sized type called `str`

- Not `&str`, but `str` on its own, is a DST

We can’t know how long the string is until runtime, meaning we can’t create a variable of type `str`, nor can we take an argument of type `str`

```rust
THIS WON'T WORK
let s1: str = "Hello there!";
let s2: str = "How's it going?";
```

Rust needs to know how much memory to allocate for any value of a particular type, and all values of a type must use the same amount of memory

- If Rust allowed us to write this code, these two `str` values would need to take up the same amount of space
- But they have different lengths

Solution: We make the types of `s1` and `s2` a `&str` rather than a `str`

- The slice data structure just stores the starting position and the length of the slice
- So although a `&T` is a single value that stores the memory address of where the `T` is located, a `&str` is *two* values: the address of the `str` and its length

As such, we can know the size of a `&str` value at compile time: it’s twice the length of a `usize`

This is the way in which dynamically sized types are used in Rust: 

- They have an extra bit of metadata that stores the size of the dynamic information

**Golden rule of dynamically sized types**: We must always put values of dynamically sized types behind a pointer of some kind

We can combine `str` with all kinds of pointers: for example, `Box<str>` or `Rc<str>`

Every trait is a dynamically sized type we can refer to by using the name of the trait

In Trait Objects:

- mentioned that to use traits as trait objects, we must put them behind a pointer, such as `&dyn Trait` or `Box<dyn Trait>` (`Rc<dyn Trait>` would work too)

### `Sized` Trait

To work with DSTs, Rust provides the `Sized` trait to determine whether or not a type’s size is known at compile time

- This trait is automatically implemented for everything whose size is known at compile time
- In addition, Rust implicitly adds a bound on `Sized` to every generic function

```rust
fn generic<T>(t: T) {
    // --snip--
}

// ---- Is treated like this: ----

fn generic<T: Sized>(t: T) {
    // --snip--
}
```

By default, generic functions will work only on types that have a known size at compile time

- However, you can use the following special syntax to relax this restriction

```rust
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

A trait bound on `?Sized` means “`T` may or may not be `Sized`” and this notation overrides the default that generic types must have a known size at compile time

The `?Trait` syntax with this meaning is only available for `Sized`, not any other traits

**Note:** we switched the type of the `t` parameter from `T` to `&T`

- Because the type might not be `Sized`, we need to use it behind some kind of pointer
- In this case, we’ve chosen a reference