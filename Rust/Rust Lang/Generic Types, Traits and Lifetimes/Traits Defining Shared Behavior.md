# Traits: Defining Shared Behavior

Similar to `interfaces` in other languages

A *trait* defines functionality in a particular type that can be shared with other types

- Used to define shared behavior in an abstract way

## Defining a Trait

A type’s behavior consists of the methods we can call on that type

**Example:** We have multiple structs that hold various kinds and amounts of text: 

- `NewsArticle` struct that holds a news story filed in a particular location
- `Tweet` that can have at most 280 characters along with metadata that indicates whether it was a new tweet, a retweet, or a reply to another tweet

We want to make a media aggregator library crate named `aggregator` that can display summaries of data that might be stored in a `NewsArticle` or `Tweet` instance

- we need a summary from each type
    - Request that summary by calling a `summarize` method on an instance

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

1. Declare a trait using the `trait` keyword and then the trait’s name
    - `Summary` in this case
    - Also declared the trait as `pub` so that crates can use trait
2. Declared the method signatures that describe the behaviors of the types that implement this trait
    - In this case is `fn summarize(&self) -> String`
3. After the method signature we use a semicolon `;`, no implementation
    - Each type implementing this trait must provide its own custom behavior for the body of the method
    - Compiler will enforce that any type that has the `Summary` trait will have the method `summarize` defined with this signature exactly

A trait can have multiple methods in its body: Method signatures are listed one per line and each end in semicolon

## Implementing a Trait on a Type

After `impl`, put the trait name we want to implement, then use the `for` keyword, then specify the name of the type we want to implement the trait for

******************Example:****************** implementation of the `Summary` trait 

- On `NewsArticle` struct: Uses the headline, the author, and the location to create the return value of `summarize`
- On `Tweet` struct: Uses username followed by the entire text of the tweet (assuming tweet len ≤= 280)

**********************Crate name:********************** aggregator

**Filename:** src/lib.rs

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

- Implementing a trait on a type is similar to implementing regular methods
    - **Difference**: After `impl`, put the trait name we want to implement, then use the `for` keyword, then specify the name of the type we want to implement the trait for
- Within the `impl` block:  put the method signatures that the trait definition has defined

Now that the library has implemented `Summary` trait on `NewsArticle` and `Tweet`, users of the crate can call the trait methods on instances of `NewsArticle` and `Tweet` in the same way we call regular methods.

- Only difference is that the user **must** bring the **trait** into scope as well as the **types**

**Example:** How a binary crate could use our `aggregator` library crate

**********************Crate name:********************** aggregator

**Filename:** src/main.rs

```rust
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize()); // Output: 1 new tweet: horse_ebooks: of course, as you probably already know, people
}
```

Other crates that depend on the `aggregator` crate can also bring the `Summary` trait into scope to implement `Summary` on their own types

**Restriction to note**: we **can** implement a trait on a type **only** if **at least one** of the **trait** or the **type** is **local** to our crate

- We **can**:
    - We can implement standard library traits like `Display` on a custom type like `Tweet` as part of our `aggregator` crate functionality because the type `Tweet` is local to our `aggregator` crate
    - Can also implement `Summary` on `Vec<T>` in our `aggregator` crate, because the trait `Summary` is local to our `aggregator` crate
- But we **can not**
    - implement external traits on external types
    - For example, we can’t implement the `Display` trait on `Vec<T>` within our `aggregator` crate
        - Because `Display` and `Vec<T>` are both defined in the standard library and aren’t local to our `aggregator` crate.

This restriction is part of a property called **coherence**, and more specifically the **orphan rule**

- Named because the parent type is not present
- Rule ensures that other people’s code can’t break your code and vice versa
    - Without the rule, two crates could implement the same trait for the same type, and Rust wouldn’t know which implementation to use.

## Default Implementations

Can be useful to have **default** **behavior** for some or all of the methods in a trait instead of requiring implementations for all methods on every type

As we implement the trait on a particular type, we **can keep** **or override** each method’s **default behavior**

******************Example:****************** Specify a default string for the `summarize` method of the `Summary` trait instead of only defining the method signature

******************Filename:****************** src/lib.rs

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

To use a default implementation to summarize instances of `NewsArticle:`

- We specify an empty `impl` block with `impl Summary for NewsArticle {}`

```rust
// ...lib.rs...
impl Summary for NewsArticle {}
// ... main.rs ...
let article = NewsArticle {...};
println!("New article available! {}", article.summarize()); // Output New article available! (Read more...)
```

Default implementations can call **other methods** in **the same trait**, even if those **other methods** **don’t** have a **default implementation**

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

To use this version of `Summary`, we only need to define `summarize_author` when we implement the trait on a type

```rust
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```

## Traits as Parameters

Use **traits** to define **functions** that **accept** many **different** **types**

Use `impl Trait` to do so

******************Example:****************** Define a `notify` function that calls the `summarize` method on its `item` parameter, which is of some type that implements the `Summary` trait

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

- Instead of a concrete type for the `item` parameter, we specify the `impl` keyword and the trait name
    - Parameter accepts any type that implements the specified trait
- In the body of `notify`, we can call any methods on `item` that come from the `Summary` trait, such as `summarize`

### Trait Bound Syntax

`impl Trait` syntax works for straightforward cases but **is actually syntax sugar** for a longer form known as a ***trait bound***

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Longer form is equivalent to the example in the previous example but more verbose

We place trait bounds with the declaration of the generic type parameter after a colon and inside angle brackets.

Can come in handy for more complex function definitions

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {  // #1
//----Equivalent----
pub fn notify<T: Summary>(item1: &T, item2: &T) {            // #2
```

- Using `impl Trait` is appropriate if we want this function to allow `item1` and `item2` to have different types  (#1)
    - (as long as both types implement `Summary`)
- If we want to force both parameters to have the same type, however, we must use a trait bound (#2)

### Specifying Multiple Trait Bounds with the `+` Syntax

We can also specify more than one trait bound

**Example:** We want `notify` to use display formatting as well as `summarize` on `item`

- Specify in the `notify` definition that `item` must implement both `Display` and `Summary`
- We can do so using the `+` syntax

```rust
pub fn notify(item: &(impl Summary + Display)) {
//----Equivalent----
pub fn notify<T: Summary + Display>(item: &T) {
```

With the two trait bounds specified, the body of `notify` can call `summarize` and use `{}` to format `item`

### Clearer Trait Bounds with `where` Clauses

Using too many trait bounds has downsides

- Each generic has its own trait bounds, so functions with multiple generic type parameters can get long
    - Makes the function signature hard to read
- For this reason, Rust has alternate syntax for specifying trait bounds inside a `where` clause after the function signature

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
//----EQUIVALENT----
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{ ... }
```

Function signature is less cluttered

- Function name, parameter list, and return type are close together, similar to a function without lots of trait bounds

## Returning Types that Implement Traits

Can also use the `impl Trait` syntax in the return position to return a value of some type that implements a trait

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("tweet contents..."),
        reply: false,
        retweet: false,
    }
}
```

- By using `impl Summary` for the return type, we specify that the `returns_summarizable` function returns some type that implements the `Summary` trait without naming the concrete type

The ability to specify a return type only by the trait it implements is **especially useful** in the context of **closures** and **iterators**

- Closures and iterators create types that only the compiler knows or types that are very long to specify
- The `impl Trait` syntax lets you concisely specify that a function returns some type that implements the `Iterator` trait without needing to write out a very long type.

**However:** Can only use `impl Trait` if function can only return a **single** type

**Example:** this code that returns either a `NewsArticle` or a `Tweet` with the return type specified as `impl Summary` won’t work

```rust
WON'T WORK!
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        return NewsArticle {...}
    } else {
        return Tweet {...}
    }
}
```

Returning either a `NewsArticle` or a `Tweet` isn’t allowed due to restrictions around how the `impl Trait` syntax is implemented in the compiler

- We can use *trait objects* that allows for values of different types

## Using Trait Bounds to Conditionally Implement Methods

Similar to method overloading

By using a **trait bound** **with** an `impl` block that **uses generic** type parameters, we can **implement methods conditionally** for types that **implement the specified traits**

**Example:** Type `Pair<T>` always implements the `new` function to return a new instance of `Pair<T>`  

But in the next `impl` block, `Pair<T>` only implements the `cmp_display` method if its inner type `T` implements the `PartialOrd`  *and*  `Display` traits 

- `PartialOrd` trait enables comparison *and* the
- `Display` trait enables printing.

recall from [“Defining Methods”](https://doc.rust-lang.org/book/ch05-03-method-syntax.html#defining-methods)  that `Self` is a type alias for the type of the `impl` block, which in this case is `Pair<T>`

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

We can also conditionally implement a trait for any type that implements another trait

*blanket implementations:* Implementations of a trait on any type that satisfies the trait bounds

**Example:** the standard library implements the `ToString` trait on any type that implements the `Display` trait. 

The `impl` block in the standard library looks similar to this code

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

Because the standard library has this blanket implementation, we can call the `to_string` method defined by the `ToString` trait on any type that implements the `Display` trait

**Example:** we can turn integers into their corresponding `String` values like this because integers implement `Display`

```rust
let s = 3.to_string();
```

Traits and trait bounds let us write code that uses generic type parameters to reduce duplication but also specify to the compiler that we want the generic type to have particular behavior

- Compiler can then use the trait bound information to check that all the concrete types used with our code provide the correct behavior