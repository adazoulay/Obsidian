# Strings: Common Methods

## Create New String

**`new()`**: Creates an empty **`String`**.

```rust
let s = String::new();
```

**`from()`**: Creates a **`String`** from a string literal or another type that implements the **`Into<String>`** trait.

```rust
let s = String::from("Hello, world!");
```

## Mutate String

**`push_str()`**: Appends a string slice `&str` to the end of the **`String`**.

```rust
let mut s = String::from("Hello");
s.push_str(", world!");
```

**`push()`**: Appends a single character to the end of the **`String`**.

```rust
let mut s = String::from("Hello");
s.push('!');
```

**`truncate()`**: Shortens the **`String`** to a specified length in bytes.

```rust
let mut s = String::from("Hello, world!");
s.truncate(5);
println!("{}", s);
```

**`replace()`**: Replaces all occurrences of a specified substring with another substring.

- Does not require `mut`

```rust
let s = String::from("Hello, world!");
let new_s = s.replace("world", "Rust");
println!("{}", new_s);
```

## String Info

**`len()`**: Returns the length of the **`String`** in bytes.

```rust
let s = String::from("Hello, world!");
println!("Length: {}", s.len());
```

**`is_empty()`**: Checks if the **`String`** is empty.

```rust
let s = String::from("Hello, world!");
println!("Is empty? {}", s.is_empty());
```

**`contains()`**: Checks if the **`String`** contains a specified substring.

```rust
let s = String::from("Hello, world!");
println!("Contains 'world': {}", s.contains("world"));
```

## Memory Management

**`capacity()`**: Returns the number of bytes the **`String`** can hold without reallocating memory

```rust
let s = String::from("Hello, world!");
println!("Capacity: {}", s.capacity());
```

**`shrink_to_fit()`**: Reduces the capacity of the **`String`** to match its length, potentially reducing memory usage.

```rust
let mut s = String::from("Hello, world!");
s.shrink_to_fit();
```

**`reserve()`**: Increases the capacity of the **`String`** to a specified amount.

```rust
let mut s = String::from("Hello, world!");
s.reserve(50);
```

## Iteration

**`chars()`**: Returns an iterator over the characters of the **`String`,** `[char]`.

```rust
let s = String::from("Hello, world!");
for c in s.chars() {
    println!("{}", c);
}
```

**`bytes()`**: Returns an iterator over the bytes of the **`String`,** `[u8]`.

```rust
let s = String::from("Hello, world!");
for b in s.bytes() {
    println!("{}", b);
}
```

**`split_whitespace()`**: Returns an iterator over substrings separated by any whitespace character, `[&str]`.

```rust
let s = String::from("Hello, world!");
for word in s.split_whitespace() {
    println!("{}", word);
}
```

## Casting

- **`parse()`**: Convert a string slice (**`&str`**) into a type that implements the **`FromStr`** trait, such as numbers,
    - Returns a **`Result`**
    - To pick type, provide a type annotation for the variable that will store the result

```rust
let input = "3.14";
let parsed_value: Result<f64, std::num::ParseFloatError> = input.parse();
```