# Hash Maps: Common Methods

## Create a new **`HashMap<K, V>`**

**`new()`**: Creates an empty **`HashMap`**.

```rust
use std::collections::HashMap;
let mut map = HashMap::new();
```

## Mutate HashMap

**`insert()`**: Inserts a key-value pair into the **`HashMap`**. If the key already exists, the previous value will be replaced.

```rust
map.insert("key1", "value1");
map.insert("key2", "value2");
```

**`remove()`**: Removes a key-value pair from the **`HashMap`**, returning the value if the key was present.

```rust
if let Some(value) = map.remove("key1") {
    println!("Removed value: {}", value);
} else {
    println!("Key not found");
}
```

**`clear()`**: Removes all key-value pairs from the **`HashMap`**.

```rust
map.clear();
```

## Access elements

**`get()`**: Retrieves a reference to the value associated with a given key, if it exists in the **`HashMap`**.

```rust
if let Some(value) = map.get("key1") {
    println!("Value: {}", value);
} else {
    println!("Key not found");
}
```

**`entry()`**: Gets the entry corresponding to the given key, allowing you to update its value or insert a new key-value pair if it doesn't exist.

- Requiers `mut`

```rust
map.entry("key3").or_insert("value3");
```

## HashMap Info

**`contains_key()`**: Checks if the **`HashMap`** contains the specified key.

```rust
if map.contains_key("key1") {
    println!("Key exists");
} else {
    println!("Key not found");
}
```

**`len()`**: Returns the number of elements in the **`HashMap`**.

```rust
println!("Number of elements: {}", map.len());
```

**`is_empty()`**: Checks if the **`HashMap`** is empty.

```rust
println!("Is empty? {}", map.is_empty());
```

## Iterate over Hashmap

**`iter()`**: Returns an iterator over the key-value pairs in the **`HashMap`**.

```rust
for (key, value) in &map {         // &map implicitly calls iter()
    println!("{}: {}", key, value);
}
```

**`keys()`**: Returns an iterator over the keys in the **`HashMap`**.

```rust
for key in map.keys() {
    println!("Key: {}", key);
}
```

**`values()`**: Returns an iterator over the values in the **`HashMap`**.

```rust
for value in map.values() {
    println!("Value: {}", value);
}
```

**`values_mut()`**: Returns an iterator over mutable references to the values in the **`HashMap`**. This allows you to modify the values in place.

```rust
for value in map.values_mut() {
    // Modify the value in some way
    *value = new_value;
}

```