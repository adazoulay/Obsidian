# Vector: Common Methods

## Create new Vector

**`new()`**: Creates an empty **`String`**

- Must specify type

```rust
let v: Vec<i32> = Vec::new();
```

`**vec!**`: Macro, creates `Vec<T>` from array of elements

```rust
let v = vec![1, 2, 3]; // Type is i32 because that’s the default integer type
```

## Mutate vectors

Requires `mut Vec<T>`

**`push()`**: Adds an element to the end of the vector.

```rust
v.push(42);
```

**`pop()`**: Removes and returns the last element in the vector, or **`None`** if the vector is empty.

- returns the removed element wrapped in an **`Option`**

```rust
if let Some(value) = v.pop() {
    println!("Popped value: {}", value);
}
```

**`insert()`**: Inserts an element at a given index, shifting all elements after the index to the right.

```rust
v.insert(1, 56); // Inserts 56 at index 1
```

**`remove()`**: Removes and returns the element at a given index, shifting all elements after the index to the left.

- returns the removed element wrapped in an **`Option`**

```rust
if let Some(value) = v.remove(2) {
    println!("Removed value at index 2: {}", value);
}
```

**`truncate()`**: Shortens the vector, keeping the first **`n`** elements and removing the rest.

```rust
v.truncate(2); // Keep the first 2 elements
```

**`clear()`**: Removes all elements from the vector, but retains its allocated capacity.

```rust
v.clear();
```

**Indexing**: Access elements in the vector using indexing or the **`get()`** method.

```rust
// Access using indexing
println!("Element at index 0: {}", v[0]);

// Access using the `get()` method
if let Some(value) = v.get(0) {
    println!("Element at index 0: {}", value);
}
```

## Vector Info

**`len()`**: Returns the number of elements in the vector.

```rust
println!("Length of vector: {}", v.len());
```

**`is_empty()`**: Checks if the vector is empty.

```rust
if v.is_empty() {
    println!("The vector is empty.");
}
```

## Memory Management

**`capacity()`**: Returns the number of elements the vector can hold without reallocating memory.

```rust
println!("Vector capacity: {}", v.capacity());
```

**`shrink_to_fit()`**: Shrinks the capacity of the vector as close as possible to its length, potentially reducing memory usage.

```rust
v.shrink_to_fit();
```

## Iteration

**`iter()`**, **`iter_mut()`**: Creates an iterator over the elements of the vector. 

- **`iter()`** creates an iterator over immutable references
- **`iter_mut()`** creates an iterator over mutable references.

```rust
for value in v.iter() {.        // OR &v implicitly calls iter()
    println!("Value: {}", value);
}

for value in v.iter_mut() {
    *value *= 2; // Modify the value in the vector
}
```

`&v`, `&mut v` 

- `&v` creates an iterator over immutable references
- `&mut v` creates an iterator over mutable references.

```rust
let v = vec![100, 32, 57];
    for i in &v {
        println!("{i}"); // Output: 100, 32, 57
    }
	 for i in &mut v {
	      *i += 50;  // Notice *i
	  }Use a `for` loop to get **mutable** references to each element in a `Vec<i32>` 
```