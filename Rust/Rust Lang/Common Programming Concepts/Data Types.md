# Data Types

Every value in Rust is of a certain *********data type*********

# Scalar Types (Primitives)

Represents a single value, predefiened size, allocated on **stack**

Can be quickly and trivially copied to make a new, independent instance if another part of code needs to use the same value in a different scope

**Four primary scalar types:**

1. integers 
2. floating-point numbers
3. Booleans 
4. characters

## Integer Types

| Length | Signed | Unsigned | U range (dec) |
| --- | --- | --- | --- |
| 8-bit | i8 | u8 | 0 ↔ 255 |
| 16-bit | i16 | u16 | 0 ↔ 65,535 |
| 32-bit | i32 | u32 | 0 ↔ $4.2*10^9$ |
| 64-bit | i64 | u64 | 0 ↔ $1.8*10^{19}$ |
| 128-bit | i128 | u128 | 0 ↔ $3.4*10^{39}$ |
| arch | isize | usize | 0 ↔ ? |

An integer is a number without a fractional component

Each variant can either be signed or unsinged and has an explicit size

- u = unsigned ↔ i = signed
    - Each unsinged variant can store: $0 \text{ to } 2^{n} -1$
    - Each signed variant can store $-(2^{n-1}) \text{ to } 2^{n-1}-1$
- `isize` and `usize` types depend on the architecture of the computer program is running on
    - Denoted in the table as “arch”: 64 / 32 bits if 64-bit / 32-bit architecture
    - The primary situation in which you’d use `isize`or `usize` is when indexing a collection.

******************************Integer Literals******************************

| Number literals | Example |
| --- | --- |
| Decimal | 98_222 |
| Hex | 0xff |
| Octal | 0o77 |
| Binary | 0b1111_0000 |
| Byte (u8 only) | b'A' |

literals can also use `_` as a visual separator to make the number easier to read

- Ex: `1_000`, will have the same value as if you had specified `1000`

**********************************Integer overflow:**********************************

Say you have a var of type `u8` (can hold vals 0↔255)

If val is set to 256 for example two things can happen:

1. When compiling in debug mode, Rust includes checks for integer overflow that cause your program to *panic* at runtime if this behavior occurs. 
2. When compiling in release mode with the `--release` flag, Rust does *not* include checks for integer overflow that cause panics. Instead, if overflow occurs, Rust performs *two’s complement wrapping. ie: loops back to 0*

## Floating-Point Types

Rust has two primitive types for floating-point numbers

`f32` and `f64`

- 32 bits and 64 bits in size, respectively.
- The default type is `f64` because on modern CPUs, it’s roughly the same speed as `f32`
 but is capable of more precision
- All floating-point types are signed.

```rust
fn main() {
    let x = 2.0; // f64
    let y: f32 = 3.0; // f32
}
```

**Numeric operations**

Rust supports basic math operations for all number types

- addition
- subtraction
- multiplication
- division
    - Integer division truncates toward zero to the nearest integer
- remainder (modulo)

```rust
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;

    // multiplication
    let product = 4 * 30;

    // division
    let quotient = 56.7 / 32.2;
    let truncated = -5 / 3; // Results in -1

    // remainder
    let remainder = 43 % 5;
}
```

## Boolean Type

Boolean type in rust has two possible values: `true` and `false`

- Booleans are one byte in size
- The Boolean type in Rust is specified using `bool`

```rust
fn main() {
    let t = true;
    let f: bool = false; // with explicit type annotation
}
```

## Char Type

Rust’s most primitive alphabetic type

```rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // with explicit type annotation
    let heart_eyed_cat = '😻';
}
```

Note: `char` literals spcified with single quotes

- 4 bytes in size
- represents a Unicode Scalar Value, which means it can represent a lot more than just ASCII
    - Accented letters; Chinese, Japanese, and Korean characters; emoji; and zero-width spaces are all valid `char` values

## Compound Types

Compound types can group multiple values into one type

Rust has 2 primive compound types: `tuples` and `arrays`

### Tuple type

A `tuple` is a general way of grouping together a number of values with a variety of types into one compound type

- Tuples have a fixed length → Once declared they cannot grow/shrink
- We create a tuple by writing a comma-separated list of values inside parentheses. Each position in the tuple has a type

```rust
fn main() {
		let tup: (i32, f64, u8) = (500, 6.4, 1);

		// We can destructure like so
    let (x, y, z) = tup;
    println!("The value of y is: {y}");

		// Or we can access val like so
		let a = tup.0;
    println!("The value of a is: {a}");
}
```

The tuple without any values has a special name, *unit*.

- This value and its corresponding type are both written `()`
- Represents an empty value or an empty return type
    - Expressions implicitly return the unit value if they don’t return any other value.

### Array type

Arrays are useful when you want your data allocated on the stack rather than the heap or when you want to ensure you always have a fixed number of elements. 

- Arrays in Rust have a fixed length
- Unlike tuple, every element in array must have the same type
- Not as flexible as `vector` type.

```rust
fn main() {
	// default way
  let a = [1, 2, 3, 4, 5];

	// This would be a good use for array since always 12 months
	let months = ["January", "February", "March", "April", "May", "June", 
								"July","August", "September", "October", "November", "December"];

	// specify type and size
	// [type ; length]
	let b: [i32; 5] = [1, 2, 3, 4, 5]; // Gives array of size 5 of type i32

	// can initialize an array to contain the same value for each element
	// [initial value ; length]
	let c = [3;5];
}
```

**Invalid Array Element Access**

```rust
...
  let a = [1, 2, 3, 4, 5];
	let val = a[9]
...
ERR
thread 'main' panicked at 'index out of bounds: the len is 5 but the index is 10', 
src/main.rs:19:19 note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

# Complex Types

Represents a multiple values, flexible size, allocated on **heap**

Lifetime can extend beyond the scope in which they are defined

## Strings

String literal: `&str`

- `let s = "hello"`
- String value is hardcoded into our program.
- Convenient but immutable

### The `String` ************************type

Manages data allocated on the heap → is able to store an amount of text that is unknown to us at compile time.

```rust
let s = String::from("hello");
```

The `::` operator allows us to namespace this particular `from` function under the `String`
 type rather than using some sort of name like `string_from`.

The following kind of String can be mutated

```rust
let mut s = String::from("hello");
s.push_str(", world!"); // push_str() appends a literal to a String
println!("{}", s); // This will print `hello, world!`
```

Question: Why can `String` be mutated but literals cannot?

- The difference is in how these two types deal with memory.