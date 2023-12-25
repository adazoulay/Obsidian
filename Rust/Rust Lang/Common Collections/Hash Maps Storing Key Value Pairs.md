# Hash Maps: Storing Key Value Pairs

[[Hash Maps  Common Methods]]

`HashMap<K, V>` stores a mapping of keys of type `K` to values of type `V` using a *hashing function*

- *hashing function* determines how it places these keys and values into memory

## Crating a New Hash Map

One way to create an empty hash map is using `new` and adding elements with `insert`

```rust
use std::collections::HashMap;
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

- Need to `use std::collections::HashMap;`
    - Not included in prelude
- Hash maps also have less support from the standard library
    - there’s no built-in macro to construct them, for example

Similarity to Vectors

- Like **vectors**, **hash maps** store their data on the **heap**
- Like vectors, hash maps are homogeneous:
    - All of the keys must have the same type, and all of the values must have the same type
    - This `HashMap` has keys of type `String` and values of type `i32`

## Accssing Values in a Hash Map

`get` method to get a value out of the hash map

```rust
use std::collections::HashMap;
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name).copied().unwrap_or(0); // score = 10
```

- `get` method returns an `Option<&V>`
    - if there’s no value for that key in the hash map, `get` will return `None`
- Handles the `Option` by calling `copied` to get an `Option<i32>` instead of `Option<&i32>`
- `unwrap_or` to set `score` to zero if `scores` doesn't have an entry for the key

## Iterating over key/value paris

Can be accomplished with a for loop

```rust
use std::collections::HashMap;
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{key}: {value}");
} 

//Output
Yellow: 50
Blue: 10
```

- This code will print each pair in an arbitrary order

## Hash Maps and Ownership

Types that **implement** the `Copy` trait the values are **copied** into the hash map

- Ex `i32`

Owned values like `String` will be moved and the hash map will be the owner of those values

```rust
use std::collections::HashMap;
let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name and field_value are invalid at this point
// using them results in error
```

- Can’t use `field_name` and `field_value` after they’ve been moved into the hash map with the call to `insert`

If we insert references to values into the hash map, the values won’t be moved into the hash map

- Values references point to must be valid for at least as long as the hash map is valid

## Updating a Hash Map

Each unique key can only have one value associated with it at a time

To change the data in a hash map, decide how to handle the case when a key already has a value assigned:

- Could replace the old value with the new value, completely disregarding the old value
- Could keep the old value and ignore the new value, only adding the new value if the key *doesn’t* already have a value
- Could combine the old value and the new value

### Overwriting a Value

`insert` to overwrite preveous value

```rust
use std::collections::HashMap;
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores); // Output: {"Blue": 25}

```

### Adding a Key and Value only if a Key isn’t Present

`entry` takes the key you want to check as a parameter. 

- Return value of the `entry` is enum called `Entry`that represents a value that might or might not exist

```rust
use std::collections::HashMap;
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

`or_insert` method on `Entry` returns 

- A mutable reference (`&mut V`) to the value for the corresponding `Entry` key if key exists
- If not, inserts the parameter as the new value for this key and returns a mutable reference to the new value

### Updating a Value Based on the Old Value

**Example:** Code that counts how many times each word appears in some text

- Use a hash map with the words as keys and increment the value to keep track of how many times we’ve seen that word
- If it’s the first time we’ve seen a word, we’ll first insert the value 0

```rust
use std::collections::HashMap;
let text = "hello world wonderful world";
let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map); // Output {"world": 2, "hello": 1, "wonderful": 1}
```

- `split_whitespace`method returns an iterator over sub-slices, separated by whitespace, of the value in `text`
- `or_insert` method returns a mutable reference (`&mut V`) to the value for the specified key
- Store mutable reference in the `count` variable → to assign to that value, we must first dereference `count` using the asterisk (`*`)
    - The mutable reference goes out of scope at the end of the `for` loop, so safe

## Hashing Functions

By default, `HashMap` uses a hashing function called *SipHash* 

- Can provide resistance to Denial of Service (DoS) attacks involving hash tables
- Not the fastest hashing algorithm available, but the trade-off for better security

If you profile your code and find that the default hash function is too slow for your purposes, you can switch to another function by specifying a different hasher.

- A *hasher* is a type that implements the `BuildHasher` trait
- [crates.io](http://crates.io) or build your own