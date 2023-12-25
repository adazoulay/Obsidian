# Debugging

`#[derive(Debug)]` trait allows us to debug

### println! maco

Can use `{:?}` and `{:#?}` instad of `{}` to print contents of structs that don’t implement `Display`

```rust
#[derive(Debug)] // Debug TRAIT
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    println!("rect1 is {:?}", rect1);
}
```

- `{:?}` output:
    - rect1 is Rectangle { width: 30, height: 50 }
- `{:#?}` output:
    - rect1 is Rectangle {
        width: 30,
        height: 50,
    }

### dbg! maco

Another way to print out a value `Debug`format is to use the `[dbg!` macro](https://doc.rust-lang.org/std/macro.dbg.html)

- Takes ownership of an expression (as opposed to `println!`, which takes a reference)
- Prints the file and line number of where that `dbg!` macro call occurs in your code along with the resultant value of that expression
- Returns ownership of the value

Note: Calling the `dbg!`macro prints to the standard error console stream (`stderr`), as opposed to `println!`, which prints to the standard output console stream (`stdout`).

```rust
#[derive(Debug)] // Still needs to implement
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale), 
        height: 50,
    };
    dbg!(&rect1);
}
```

Output:

[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}

- We can put `dbg!` around the expression `30 * scale` and, because `dbg!`  returns ownership of the expression’s value, the `width`field will get the same value as if we didn’t have the `dbg!`
- We don’t want `dbg!` to take ownership of `rect1`, so we use a reference to `rect1`in the next call. Here’s what the output of this example looks like: