# Common Attributes and Traits

**Deriving traits**: Some common traits can be automatically derived for your types using the **`#[derive(...)]`** attribute. 

This is a convenient way to implement the following traits without having to write the implementation yourself: (* somtimes)

- **`Debug`**
- **`Clone`**
- **`Copy`**
- **`Default`**
- **`PartialEq` ***
- **`Eq` ***
- **`PartialOrd` ***
- **`Ord` ***

```rust
#[derive(Debug, Clone, PartialEq, Default)]
struct MyStruct {
    field: i32,
}
```

# Printing

**`Debug`**: Enables pretty-printing of the type for debugging purposes.

- Can usually be derived automatically, but you can implement it manually if you want to customize the debug output.

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

let p = Point { x: 3, y: 4 };
println!("{:?}", p);
```

**`Display`**:  Used for user-facing, nicely formatted output

- Implementing **`Display`** requires you to define the **`fmt`** function, which specifies how the type should be displayed to the user.

```rust
use std::fmt;

struct Person {
    name: String,
    age: u32,
}

impl fmt::Display for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{} ({})", self.name, self.age)
    }
}

fn main() {
    let person = Person {
        name: String::from("Alice"),
        age: 30,
    };
    println!("{}", person); // Output: Alice (30)
}
```

# Enstantiation

**`Clone`**: Allows the type to be cloned, creating a new instance with the same values.

- Can usually be derived automatically, but you can implement it manually for custom cloning behavior.

```rust
#[derive(Clone)]
struct Point {
    x: i32,
    y: i32,
}

let p1 = Point { x: 3, y: 4 };
let p2 = p1.clone();
```

********`Copy`:** A trait for types with simple bitwise copy semantics, allowing implicit duplication without calling any function.

- Can be derived automatically, but you need to ensure that your type fulfills the requirements for being **`Copy`**
    1. The type must implement the **`Clone`** trait.
    2. All of its components (fields) must also be **`Copy`**.
    3. The type should not have any destructors (i.e., no **`Drop`** trait implementation).

```rust
#[derive(Debug, Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = p1; // No need to call .clone() or any other function; p2 is a bitwise copy of p1
    println!("{:?}", p1); // p1 is still valid and can be used
}
```

**`Default`**: Provides a default value for the type, which can be used to initialize instances without explicitly specifying all field values.

- Can be derived automatically, but you need to ensure that your type fulfills the requirements for being

```rust
#[derive(Default)]
struct Point {
    x: i32,
    y: i32,
}

let p: Point = Default::default(); // x: 0, y: 0
```

# Comparison

**`PartialEq`**: Enables comparison for equality (==) and inequality (!=) of instances of the type.

- Can be derived automatically for many cases, but may need manual implementation for complex custom types.

```rust
#[derive(PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

let p1 = Point { x: 3, y: 4 };
let p2 = Point { x: 3, y: 4 };
println!("{}", p1 == p2); // true
```

**`Eq`**: Indicates that instances of the type have a total equivalence relation, which means that in addition to the **`PartialEq`** requirements, all instances are equal to themselves, and inequality is the negation of equality.

- Can be derived automatically for many cases, but may need manual implementation for complex custom types

```rust
#[derive(Debug, PartialEq)]
struct Person {
    name: String,
    age: u32,
}

impl Eq for Person {}

fn main() {
    let p1 = Person { name: String::from("Alice"), age: 30 };
    let p2 = Person { name: String::from("Alice"), age: 30 };

    println!("p1 == p2: {}", p1 == p2);
}
```

**`PartialOrd`**: Enables comparison for ordering (<, >, <=, >=) of instances of the type.

- Can be derived automatically for simple cases, but may need manual implementation for complex custom types or custom ordering logic.
- compare the fields lexicographically, shortcircuits at first inequality

```rust
use std::cmp::Ordering;

#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl PartialOrd for Point {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        Some(self.x.cmp(&other.x))
    }
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 2, y: 3 };

    println!("p1 < p2: {}", p1 < p2);
}
```

**`Ord`**: Indicates that instances of the type have a total order relation, which means that in addition to the **`PartialOrd`** requirements, all instances are comparable to all other instances.

- Can be derived automatically for simple cases, but may need manual implementation for complex custom types or custom ordering logic.

```rust
use std::cmp::Ordering;

#[derive(Debug, PartialEq, Eq, PartialOrd)]
struct Point {
    x: i32,
    y: i32,
}

impl Ord for Point {
    fn cmp(&self, other: &Self) -> Ordering {
        self.x.cmp(&other.x).then_with(|| self.y.cmp(&other.y))
    }
}

fn main() {
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 1, y: 3 };

    println!("p1 < p2: {}", p1 < p2);
}
```

# Casting

**`From`**: A trait for converting a value of one type into a value of another type. Implementing **`From`** for a type allows you to use the **`from()`** function to perform the conversion.

- Requires manual implementation.

```rust
struct Person {
    name: String,
}

impl From<&str> for Person {
    fn from(name: &str) -> Self {
        Person { name: name.to_string() }
    }
}

fn main() {
    let p = Person::from("Alice");
}
```

**`Into`**: A trait for converting a value into another type. This trait is the reciprocal of **`From`**. If you have implemented **`From`**, you get **`Into`** for free.

- Requires manual implementation.

```rust
fn takes_string(s: String) {
    // Do something with s
}

let p = Person { name: "Alice".to_string() };
takes_string(p.into()); // Converts p into a String
```

# References

**`Deref`**: A trait for overloading dereference operator `*****`. Allows you to define how to get a reference to the value inside a custom smart pointer type.

- Requires manual implementation.

```rust
use std::ops::Deref;

struct Wrapper {
    value: String,
}

impl Deref for Wrapper {
    type Target = String;

    fn deref(&self) -> &String {
        &self.value
    }
}

fn main() {
    let mut w = Wrapper { value: "hello".to_string() };
    w.push_str(" world");
    println!("{}", *w); // Output: "hello world"
}
```

**`DerefMut`**: A trait for overloading mutable dereference operator `*****`. Allows you to define how to get a mutable reference to the value inside a custom smart pointer type.

- Requires manual implementation.

```rust
use std::ops::DerefMut;

struct MutableWrapper {
    value: String,
}

impl DerefMut for MutableWrapper {
    fn deref_mut(&mut self) -> &mut String {
        &mut self.value
    }
}

fn main() {
    let mut mw = MutableWrapper { value: "hello".to_string() };
    
    // Access the mutable reference of the underlying String using DerefMut
    let value_mut_ref = mw.deref_mut();
    value_mut_ref.push_str(" world");
    
    println!("{}", mw.value); // Output: "hello world"
}
```

**Example** with Both:

- **`MutableWrapper`** struct containing a **`String`**
    - We implemented the **`DerefMut`** trait for **`MutableWrapper`**, which allows us to get a mutable reference to the underlying **`String`**
- In the **`main`** function, we use the **`deref_mut`** method to obtain a mutable reference to the **`String`** and then call **`push_str`** on it to modify the value
- Note: This example only demonstrates the use of the **`DerefMut`** trait and does not provide the seamless method delegation that you would get with the combination of **`Deref`** and **`DerefMut`**

```rust
use std::ops::{Deref, DerefMut};

struct Wrapper {
    value: String,
}

impl Deref for Wrapper {
    type Target = String;

    fn deref(&self) -> &String {
        &self.value
    }
}

impl DerefMut for Wrapper {
    fn deref_mut(&mut self) -> &mut String {
        &mut self.value
    }
}

fn main() {
    let mut w = Wrapper { value: "hello".to_string() };
    w.push_str(" world");
    println!("{}", *w); // Output: "hello world"
}
```

**`Drop`**: A trait provided by Rust to clean up an object when it goes out of scope. The Drop trait provides a **`drop`** method which is called automatically when an object goes out of scope. This is typically used to release resources that are not managed by Rust's memory system, such as network connections, files or heap-allocated memory.

- Requires manual implementation

```rust
struct HasDrop;

impl Drop for HasDrop {
    fn drop(&mut self) {
        println!("Dropping HasDrop!");
    }
}

fn main() {
    let _x = HasDrop;
    // Do other stuff
} // `drop` is called here, at the end of the block.
```

# Iteration

**`Iterator`**: A trait that provides a way to process a sequence of elements one at a time.

- Cannot be derived automatically, it needs to be implemented manually.
- Requires implementing the **`next`** method, which defines how to move onto the next element in the sequence, returns an `Option`

```rust
struct Counter {
    count: u32,
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count <= 5 {
            Some(self.count)
        } else {
            None
        }
    }
}

fn main() {
    let mut counter = Counter { count: 0 };
    
    for count in counter {
        println!("{}", count);
    }
}
```