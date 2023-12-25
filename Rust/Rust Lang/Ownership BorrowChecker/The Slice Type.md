# The Slice Type

**Slices** let you reference a contiguous sequence of elements in a collection rather than the whole collection

 A slice is a kind of reference, so it does not have ownership

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

- `first_word` takes `&String` as a parameter
    - We don’t want ownership
- Need to go through the `String` element by element and check whether a value is a space
    - Convert our `String` to an array of bytes using the `as_bytes` method
- Next, we create an iterator over the array of bytes using the `iter` method
    - `enumerate` wraps the result of `iter` and returns each element as part of a tuple instead
- In the `for` loop, we specify a pattern that has `i` for the index in the tuple and `&item`
 for the single byte in the tuple
    - Because we get a reference to the element from `.iter().enumerate()`, we use `&` in the pattern
- Inside the `for` loop, we search for the byte that represents the space by using the byte literal syntax
    - If we find a space `b’ ‘`, we return the position. Otherwise, we return the length of the string by using `s.len()`

## String Slices

A *******string slice******* is a reference to a part of a `String`

```rust
let s = String::from("hello world");
let hello = &s[0..5];
let world = &s[6..11];
```

- The notation for a slice is `[starting_index..ending_index]`
    - `starting_index` is the first position in the slice
    - `ending_index` is one more than the last position in the slice.
- Internally, the slice data structure stores the starting position and the length of the slice, which corresponds to `ending_index` minus `starting_index`
    - in the case of `let world = &s[6..11];`
        - `world`would be a slice that contains a pointer to the byte at index 6 of `s`
         with a length value of `5`
    
    .
    
    ![[trpl04-06.svg]]
    
    Using the `Range` operator, we can rewrite `first_word` to return a slice
    
    The type that signifies a “string slice” is `&str`
    
    ```rust
    fn first_word(s: &String) -> &str {
        let bytes = s.as_bytes();
    
        for (i, &item) in bytes.iter().enumerate() {
            if item == b' ' {
                return &s[0..i];
            }
        }
    
        &s[..]
    }
    ```
    
- We get the index for the end of the word the same way we did above
    - By looking for the first occurrence of a space
- When we find a space, we return a string slice using the start of the string and the index of the space as the starting and ending indices.
- When we call `first_word`, we get back a single value that is tied to the underlying data
    - The value is made up of a reference to the starting point of the slice and the number of elements in the slice

### String Literals as Slices

Now that we understand slices, we can properly understand string literals

```rust
let s = "Hello, world!";
```

The type of `s` here is `&str`: it’s a slice pointing to that specific point of the binary. This is also why string literals are immutable; `&str` is an immutable reference.

### String Slices as Parameters

Knowing that you can take slices of literals and `String` values leads us to one more improvement on `first_word`, and that’s its signature:

```rust
fn first_word(s: &String) -> &str {
to
fn first_word(s: &str) -> &str {
```

More experienced Rustacean would write the second signature 

- Allows us to use the same function on both `&String` values and `&str` values.