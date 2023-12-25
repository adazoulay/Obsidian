# Strings: Storing UTF-8 Encoded Text

[[Strings  Common Methods]]

When referring to “strings” in Rust either the `String` or the string slice `&str` types

## Creating a New String

`String` is actually implemented as a wrapper around a vector of bytes with some extra guarantees, restrictions, and capabilities

- Many of the operations available with `Vec<T>` are available with `String`

```rust
let mut s = String::new();
```

creates a new empty string called `s`

Often, we’ll have some initial data that we want to start the string with 

- For that, we use the `to_string` method
    - Available on any type that implements the `Display` trait, as string literals do.

```rust
let data: &str = "initial contents";
let s: String = data.to_string();

// the method also works on a literal directly:
let s: String = "initial contents".to_string();
```

- This code creates a `String` containing `initial contents`

 `String::from` to create a `String` from a string literal

```rust
let s = String::from("initial contents");
```

Because strings are used for so many things, we can use many different generic APIs for strings

- Can be redundant

## Updating a String

A `String` can grow in size and its contents can change, just like the contents of a `Vec<T>`

- Can `push` and `push_str`
- Can also use `+` operator or the `format!` macro to concatenate `String` values.

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

- Now `s` contains `foobar`

`push_str` method takes a string slice `&str` because we don’t necessarily want to take ownership of the parameter

- For example, in the following code we want to be able to use `s2` after appending its contents to `s1`

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {s2}");
```

- If `push_str` took ownership of `s2`, we wouldn’t be able to print its value on the last line
    - However, this code works as we’d expect

`push` method takes a single character as a parameter and adds it to the `String`

```rust
let mut s = String::from("lo");
s.push('l');
```

- `s`will contain `lol`

## Concatenation with the `+` Operator or the `format!` Macro

`+` operator to combine two existing Strings

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
```

- `s3`will contain `Hello, world!`
- `s1` is no longer valid after the addition,
    - The reason we use a reference to `s2`, has to do with the signature of the method that’s called when we use the `+` operator.

The `+` operator uses the `add` method, whose signature looks something like this:

```rust
fn add(self, s: &str) -> String {
```

- First, `s2` has an `&`, meaning that we’re adding a *reference* of the second string to the first string
    - This is because of the `s` parameter in the `add` function: we can only add a `&str` to a `String`

**Note**: the type of `&s2` is `&String`, not `&str`, as specified in the second parameter to `add`

- The reason we’re able to use `&s2` in the call to `add` is that the compiler can *coerce* the `&String` argument into a `&str`
- When we call the `add` method, Rust uses a *deref coercion*, which here turns `&s2` into `&s2[..]`
- Second, the `add` signature  takes ownership of `self`, because `self` does *not* have an `&`
    - This means `s1` will be moved into the `add` call and will no longer be valid after that

**Problem**: If we need to concatenate multiple strings, the behavior of the `+` operator gets unwieldy

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

- Sets `s` to `tic-tac-toe`
    - With all of the `+` and `"` characters, it’s difficult to see what’s going on

`format!` macro for more complicated string combining

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{s1}-{s2}-{s3}");
```

- Sets `s` to `tic-tac-toe`
- `format!` macro uses references so that this call doesn’t take ownership of any of its parameters.

## Indexing into Strings

In many other programming languages, accessing individual characters in a string by referencing them by index is a valid and common operation

```rust
THIS WON'T WORK
let s1 = String::from("hello");
let h = s1[0];   // error: `String` cannot be indexed by `{integer}`
```

### Internal Representation

A `String` is a wrapper over a `Vec<u8>`

```rust
let hello = String::from("Hola");
```

In this case, `len` will be 4, which means the vector storing the string “Hola” is 4 bytes long

- Each of these letters takes 1 byte when encoded in UTF-8

However, in this example:

```rust
let hello = String::from("Здравствуйте"); // len looks like it should be 12
```

- `len` is Actually 24
- Because each Unicode scalar value in that string takes 2 bytes of storage
- Therefore, an index into the string’s bytes will not always correlate to a valid Unicode scalar value

### Bytes and Scalar Values and Grapheme Clusters! Oh My!

There are actually three relevant ways to look at strings from Rust’s perspective:

- Bytes
- Scalar values
- Grapheme clusters
    - the closest thing to what we would call *letters*

## Slicing Strings

Indexing into a string is often a bad idea because it’s not clear what the return type of the string-indexing operation should be

- A byte value, a character, a grapheme cluster, or a string slice

Rather than indexing using `[]` with a single number, you can use `[]` with a range `x..=y` to create a string slice containing particular bytes

```rust
let hello = "Здравствуйте";
let s = &hello[0..4];
```

Here, `s` will be a `&str` that contains the first 4 bytes of the string 

- Earlier, we mentioned that each of these characters was 2 bytes, which means `s` will be `Зд`

If we were to try to slice only part of a character’s bytes with something like `&hello[0..1]` Rust would panic

## Methods for Iterating Over Strings

Best way to operate on pieces of strings: be explicit about weather you want **characters** or **bytes**

- `chars` method: for individual Unicode scalar values
- `bytes` method: to get each raw byte

```rust
for c in "Зд".chars() {
    println!("{c}"); // Output: З, д
}

for b in "Зд".bytes() {
    println!("{b}"); // Output: 208, 151, 208, 180
} 
```