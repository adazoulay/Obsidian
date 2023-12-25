# Generic Data Types

*generics*: abstract stand-ins for concrete types or other properties

We use generics to create definitions for items like function signatures or structs, which we can then use with many different concrete data types

## In Function Defenitions

When defining a function that uses generics, we place the generics in the signature of the function where we would usually specify the data types of the parameters and return value

**Example:** two functions that both find the largest value in a slice. We'll then combine these into a single function that uses generics.

```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    return largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    return largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
}
```

To parameterize the types in a new single function, we need to name the type parameter, just as we do for the value parameters to a function

- `T` is the convention

To define the generic `largest` function:

- Place type name declarations inside angle brackets, `<>`, between the name of the function and the parameter list

```rust
fn largest<T>(list: &[T]) -> &T {
```

We read this definition as:

- function `largest` is generic over some type `T`
- This function has one parameter named `list`, which is a slice of values of type `T`
- The `largest` function will return a reference to a value of the same type `T`

**Example:** The combined `largest` function definition using the generic data type in its signature

Can be called with a slice of either `i32` or `char` values

```rust
WON'T WORK YET!
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {    <-- Error: binary operation `>` cannot be applied to type `&T`
            largest = item;
        }
    }

    return largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest(&char_list);
    println!("The largest char is {}", result);
} 
```

- body of `largest` won’t work for all possible types that `T` could be
    - Because we want to compare values of type `T` in the body, we can only use types whose values can be ordered
- To enable comparisons, the standard library has the `std::cmp::PartialOrd` trait that you can implement on types

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {      // Now works
```

## In Struct Definitions

We can also define structs to use a generic type parameter in one or more fields using the `<>` syntax

1. Declare the name of the type parameter inside angle brackets just after the name of the struct
2. Then we use the generic type in the struct definition where we would otherwise specify concrete data types.

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

**Note:** because we’ve used only one generic type to define `Point<T>`, this definition says that the `Point<T>` struct is generic over some type `T`

- Hence fields `x` and `y` must *both* have that same type

**Example:** If they are not, we get a missmatch type error:

```rust
THIS WON'T WORK!
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 }; // error: type mismatch i32 and f64
}
```

### Multiple Generic Type Parameters

**Example:** We change the definition of `Point` to be generic over types `T` and `U` 

- Where `x` is of type `T` and `y` is of type `U`

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 }; 
    let both_float = Point { x: 1.0, y: 4.0 }; 
    let integer_and_float = Point { x: 5, y: 4.0 }; // Point<i32, f64>
}
```

- Can use as many generic type parameters in a definition as you want
    - Using more than a few makes your code hard to read.

If you're finding you need lots of generic types in your code, it could indicate that your code needs restructuring into smaller pieces.

## In Enum Definitions

```rust
enum Option<T> {
    Some(T),
    None,
}
```

`Option<T>` enum is generic over type `T` and has two variants: 

1. `Some`, which holds one value of type `T`
2. `None` variant that doesn’t hold any value.

Enums can use multiple generic types as well

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result` enum is generic over two types, `T` and `E`, and has two variants:

1. `Ok`, which holds a value of type `T`
2. `Err`, which holds a value of type `E`

## In Method Definitions

We can implement methods on structs and enums and use generic types in their definitions, too

******************Example:****************** `Point<T>` struct a method named `x` implemented on it

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {       // <T> after impl
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };
    println!("p.x = {}", p.x());
}
```

**Note:** Need to declare `T` just after `impl` so we can use `T` to specify that we’re implementing methods on the type `Point<T>`

- By declaring `T` as a generic type after `impl`, Rust knows type in the angle brackets in `Point` is a generic type rather than a concrete type

### Constraints in Method Definitions

We can also specify constraints on generic types when defining methods on the type

**Example:** implement methods only on `Point<f32>` instances rather than on `Point<T>` instances with any generic type

Done by specifying type in `<>` after struct name

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

- This code means the type `Point<f32>` will have a `distance_from_origin` method
    - Other instances of `Point<T>` where `T` is not of type `f32` will not have this method defined

Generic type parameters in a struct definition aren’t always the same as those you use in that same struct’s method signatures

**Example:**

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y); // Output: p3.x = 5, p3.y = c
}
```

- Uses
    - Generic types `X1` `Y1` for the `Point` struct
    - `X2` `Y2` for the `mixup` method signature to make the example clearer
- The method creates a new `Point` instance with:
    - `x` value from the `self` `Point` (of type `X1`)
    - `y` value from the passed-in `Point` (of type `Y2`).

## Performance of Code using Generics

Generic types won't make your program run any slower than it would with concrete types

Rust accomplishes this by performing *monomorphization* of code using generics at compile time

- *Monomorphization:* The process of turning generic code into specific code by filling in the concrete types that are used when compiled
- Compiler does the opposite of the steps we used to create the generic function

```rust
let integer = Some(5);
let float = Some(5.0);
```

monomorphization → Compiler identifies two kinds of `Option<T>` one is `i32` and the other is `f64`

.