# Iterators: Processing Series of Items

Iterator pattern allows you to perform some task on a sequence of items in turn

- Responsible for the logic of iterating over each item and determining when the sequence has finished

In Rust, iterators are *lazy:* They have no effect until you call methods that consume the iterator to use it up
When using iterators, you don’t have to reimplement that logic yourself.

****************Example:**************** Creates an iterator over the items in the vector `v1` by calling the `iter` method defined on `Vec<T>`

```rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();
```

The iterator is stored in the `v1_iter` variable
Here we separate the creation of the iterator from the use of the iterator in the `for` loop

- When the `for` loop is called using the iterator in `v1_iter`, each element in the iterator is used in one iteration of the loop, which prints out each value.

```rust
for val in v1_iter {
    println!("Got: {}", val);
}
```

## The `Iterator` Trait and the `next` Method

All iterators implement a trait named `Iterator` that is defined in the standard library. Here is the definition:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations omitted
}
```

**Note:** New Syntax

- `type Item` and `Self::Item`, which are defining an *associated type* with this trait
- TLDR: This code says implementing the `Iterator` trait requires that you also define an `Item` type, and this `Item` type is used in the return type of the `next` method. In other words, the `Item` type will be the type returned from the iterator.

The `Iterator` trait only requires implementors to define one method: the `next` method, which returns one item of the iterator at a time wrapped in `Some` and, when iteration is over, returns `None`

We can call the `next` method on iterators directly

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

**Note:** We needed to make `v1_iter` mutable:

- Calling the `next` method on an iterator changes internal state that the iterator uses to keep track of where it is in the sequence
- In other words, this code *consumes*, or uses up, the iterator. Each call to `next` eats up an item from the iterator
- We didn’t need to make `v1_iter` mutable when we used a `for` loop because the loop took ownership of `v1_iter` and made it mutable behind the scenes

**Note:** The values we get from the calls to `next` are immutable references to the values in the vector

- The `iter` method produces an iterator over immutable references
- If we want to create an iterator that takes ownership of `v1` and returns owned values, we can call `into_iter` instead of `iter`
- Similarly, if we want to iterate over mutable references, we can call `iter_mut` instead of `iter`.

## Methods that Consume the Iterator

`Iterator` trait has a number of different methods with default implementations provided by the standard library

Some of these methods call the `next` method in their definition, which is why you’re required to implement the `next` method when implementing the `Iterator` trait

***consuming adaptors:*** Methods that call `next`

- Calling them uses up the iterator

**Example:** `sum` method

- Takes ownership of the iterator and iterates through the items by repeatedly calling `next`, thus consuming the iterator

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();
    let total: i32 = v1_iter.sum();
    assert_eq!(total, 6);
}
```

We aren’t allowed to use `v1_iter` after the call to `sum` because `sum` takes ownership of the iterator we call it on

## Methods that Produce Other Iterators

***Iterator adaptors:*** Methods defined on the `Iterator` trait that don’t consume the iterator. Instead, they produce different iterators by changing some aspect of the original iterator

**Example:** `map` method

- Takes a closure to call on each item as the items are iterated through
- Returns a new iterator that produces the modified items

The closure here creates a new iterator in which each item from the vector will be incremented by 1

```rust
let v1: Vec<i32> = vec![1, 2, 3];
v1.iter().map(|x| x + 1); // warning: unused `Map` that must be used
```

Code produces a warning: iterator adaptors are lazy, and we need to consume the iterator here

- To fix this warning and consume the iterator, we’ll use the `collect` method
    - Consumes the iterator and collects the resulting values into a collection data type.

```rust
let v1: Vec<i32> = vec![1, 2, 3];
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
assert_eq!(v2, vec![2, 3, 4]);
```

Because `map` takes a closure, we can specify any operation we want to perform on each item

- Great example of how closures let you customize some behavior while reusing the iteration behavior that the `Iterator` trait provides

Can chain multiple calls to iterator adaptors to perform complex actions in a readable way. 
But because all iterators are lazy, you have to call one of the consuming adaptor methods to get results from calls to iterator adaptors.

## Using Clojures that Capture Their Environment

Many iterator adapters take closures as arguments, and commonly the closures we’ll specify as arguments to iterator adapters will be closures that capture their environment

**Example:** `filter` method that takes a closure
The closure gets an item from the iterator and returns a `bool`

- If closure returns `true` → value included  — If closure returns `false` → value won’t be included

`filte`  closure captures the `shoe_size` variable from its environment to iterate over a collection of `Shoe` struct instances

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn filters_by_size() {
        let shoes = vec![
            Shoe {size: 10, style: String::from("sneaker")},
            Shoe {size: 13, style: String::from("sandal")},
            Shoe {size: 10, style: String::from("boot")},
        ];

        let in_my_size = shoes_in_size(shoes, 10);

        assert_eq!(
            in_my_size,
            vec![
                Shoe {size: 10, style: String::from("sneaker")},
                Shoe {size: 10, style: String::from("boot")},
            ]
        );
    }
}
```

`shoes_in_size` function takes ownership of a vector of shoes and a shoe size as parameters. It returns a vector containing only shoes of the specified size

- In method body, we call `into_iter` to create an iterator that takes ownership of the vector
- Then we call `filter` to adapt that iterator into a new iterator that only contains elements for which the closure returns `true`
- Closure captures the `shoe_size` parameter from the environment and compares the value with each shoe’s size, keeping only shoes of the size specified
- Finally, calling `collect` gathers the values returned by the adapted iterator into a vector that’s returned by the function.

## Loops vs Iterators: Performance

Iterators slightly faster