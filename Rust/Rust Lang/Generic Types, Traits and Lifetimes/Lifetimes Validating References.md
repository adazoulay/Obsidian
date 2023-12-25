# Lifetimes: Validating References

*Lifetimes* ensure that references are valid as long as we need them to be.

every reference in Rust has a *lifetime*

- Which is the scope for which that reference is valid

Most of the time, **lifetimes** are **implicit and inferred**, just like most of the time, types are inferred

- Only must annotate types when multiple types are possible
- In a similar way, we must annotate lifetimes when the lifetimes of references could be related in a few different ways

Rust **requires** us to **annotate** the relationships using **generic lifetime parameters** to **ensure** the actual **references** used at runtime **will definitely be valid**.

## Preventing Dangling References with Lifetimes

Main aim of lifetimes is to **prevent** *dangling references*

**Note:** The next examples declare variables without giving them an initial value, so the variable name exists in the outer scope. 

- At first glance, this might appear to be in conflict with Rust’s having no null values.
- However, if we try to use a variable before giving it a value, we’ll get a compile-time error

### The Borrow Checker

*borrow checker* compares scopes to determine whether all borrows are valid

- At compile time, Rust compares size of two lifetimes and sees that `r` has a lifetime of `'a` but that it refers to memory with a lifetime of `'b`
- The program is rejected because `'b` is shorter than `'a`
    - The subject of the reference doesn’t live as long as the reference.

```rust
THIS WON'T WORK
// r lifetime:'a ; x lifetime'b
fn main() {
    let r;              // ---------+-- 'a
                        //          |
    {                   //          |
        let x = 5;      // -+-- 'b  |
        r = &x;         //  |       |   // <-- error: borrowed value does not live long enough
    }                   // -+       |
                        //          |
    println!("r{}", r); //          |
}                       // ---------+    
```

- `x` doesn’t “live long enough.”
    - The reason is that `x` will be out of scope when the inner scope ends
    - But `r` is still valid for the outer scope; because its scope is larger, we say that it “lives longer.”
    - Won’t compile: Value of `r` is referring to has gone out of scope before we try to use it → dropped
    - If Rust allowed this code to work, `r` would be referencing memory that was deallocated when `x` went out of scope, and anything we tried to do with `r`
     wouldn’t work correctly.

This code works:

Here, `x` has the lifetime `'b`, which in this case is larger than `'a`

```rust
fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```

- Means `r` can reference `x` because Rust knows that the reference in `r` will always be valid while `x` is valid.

## Generic Lifetimes in Functions

**Example:** function that returns the longer of two string slices

- Takes two `&str` and returns a single `&str`

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

After we implement, should return “abcd”

**Note:** We want the function to take string slices, which are references, rather than strings, because we don’t want the `longest` function to take ownership of its parameter

If we try to implement the `longest` function as shown it won’t compile.

```rust
WON'T WORK! 
fn longest(x: &str, y: &str) -> &str {     // err: missing lifetime specifier
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

- Help text reveals that the return type needs a generic lifetime parameter on it
- Rust can’t tell whether the reference being returned refers to `x` or `y`
    - we don’t know whether the `if` case or the `else` case will execute
    - Also don’t know the concrete lifetimes of the references that will be passed in, so we can’t look at the scopes to determine if returned reference will always be valid

## Lifetime Annotation Syntax

Names of lifetime parameters must start with an apostrophe (`'`) and are usually all lowercase and very short

Lifetime annotations don’t change how long a reference lives, but describe relationships of the lifetimes of multiple references to each other

- fn can accept any type when signature specifices generic param ↔ functions can accept references with any lifetime by specifying generic lifetime parameter

**Note:** Most people use the name `'a` for the first lifetime annotation

We place lifetime parameter annotations after the `&` of a reference, using a space to separate the annotation from the reference’s type

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

One lifetime annotation by itself doesn’t have much meaning

- annotations are meant to tell Rust how generic lifetime parameters of multiple references relate to each other

### Lifetime Annotation in Function Signatures

To use lifetime annotations in function signatures: 

- Need to declare the generic *lifetime* parameters inside angle brackets between the function name and the parameter list
    - Just as we did with generic *type* parameters.

We want the signature to express the following constraint: the returned reference will be valid as long as both the parameters are valid. 

**Note:** Not changing the lifetimes of any values passed or returns. Instead: specifying that borrow checker should reject values that don’t adhere to constraints

- `longest` doesn’t need to know how long `x` and `y` live, only that some scope can be substituted for `'a` that will satisfy this signature

Annotating lifetimes functions goes in the signature, not in body.

- Lifetime annotations become part of the function contract. Like the types in the signature

**Example:** We’ll name the lifetime `'a` and then add it to each reference to `longest`

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

- Function signature now tells Rust that for some lifetime `'a`, the function takes two parameters
    - Both of which are string slices that live at least as long as lifetime `'a`
- The function signature also tells Rust that the string slice returned from the function will live at least as long as lifetime `'a`

**Means** that the **lifetime** of the **reference returned** by the `longest` function is **the same** as the **smaller of** the **lifetimes** of the **values referred to** by the function **arguments**

```rust
fn main() {
    let string1 = String::from("long string is long");
    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
    // println!("The longest string is {}", result); THIS WOUNDN'T WORK
}
```

- `string1` is valid until the end of the outer scope, `string2` is valid until the end of the inner scope → `result` references something that is valid until the end of the inner scope

## Thinking in Terms of Lifetimes

Way in which you need to specify lifetime parameters depends on what your function is doing

**example:** if we change the implementation of the `longest` function to always return the first parameter rather than the longest string slice

- Wouldn’t need to specify a lifetime on the `y` parameter

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

When returning a reference from a function, the lifetime parameter for the return type needs to match the lifetime parameter for one of the parameters

- If the reference returned does *not* refer to one of the parameters, it must refer to a value created within this function

```rust
THIS WON'T WORK
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
THIS DOES WORK!
fn longest<'a>(x: &str, y: &str) -> String {
    let result: String = String::from("really long string") + x; // + x added
    return result;
}
```

Even though lifetime parameter `'a`  specified for the return type, this implementation fails to compile because return value lifetime is not related to the lifetime of the parameters at all

## Lifetime Annotations in Struct Definitions

We can define structs to hold references 

To do so, we need to add a lifetime annotation on every reference in the struct’s definition

As with generic data types: Declare the name of the generic lifetime parameter inside `<>` after the struct name so we can use the lifetime parameter in the body of struct definition

**Example:**

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

- struct has the single field `part` that holds a string slice, which is a reference
- `<'a>` annotation means an instance of `ImportantExcerpt` can’t outlive the reference it holds in its `part` field
- Data in `novel`exists before the `ImportantExcerpt` instance is created
- `novel` doesn’t go out of scope until after the `ImportantExcerpt` goes out of scope, so the reference in the `ImportantExcerpt` instance is valid

## Lifetime Elision

Every reference has a lifetime and that you need to specify lifetime parameters for functions or structs that use references

However, this example compiles without lifetime annotations

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    return &s[..]
}
```

The reason this function compiles without lifetime annotations is historical: 

- In early versions (pre-1.0) of Rust, this code wouldn’t have compiled because every reference needed an explicit lifetime.

At that time, the function signature would have been written like this:

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

Rust team found that Rust programmers were entering the same lifetime annotations over and over in particular situations. 

- These situations were predictable and followed a few deterministic patterns

*lifetime elision rules:* **Implicit Patterns** programmed into Rust’s analysis of references 

- Not rules for programmers to follow
    - A set of particular cases that the compiler will consider, where you don’t need to write the lifetimes explicitly

### *lifetime elision,* Three Rules:

1. The compiler assigns a lifetime parameter to each parameter that’s a reference. 
    1. A function with one parameter gets one lifetime parameter: `fn foo<'a>(x: &'a i32)`; 
    2. A function with **two** parameters gets two **separate** **lifetime** **parameters**: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`; …
2. If there is **exactly one input lifetime parameter**, that lifetime is assigned to **all output lifetime parameters**: 
    1. `fn foo(x: &i32) -> i32`  ↔  `fn foo<'a>(x: &'a i32) -> &'a i32`.
3. The third rule is that, if there are **multiple input lifetime parameters**, but **one** of them is `&self` or `&mut self` because this is a **method**, the lifetime of `self` is **assigned to** **all** **output** lifetime parameters. 
    1. Makes methods much nicer to read and write because fewer symbols are necessary.

**Example1:** Pretend we are compiler

Apply these rules to figure out the lifetimes of the references in the signature of the `first_word` function

```rust
fn first_word(s: &str) -> &str {
```

1. Compiler applies the first rule:
    - Specifies that each parameter gets its own lifetime. We’ll call it `'a`

```rust
fn first_word<'a>(s: &'a str) -> &str {
```

1. Compier applies second rule because there is exactly one input lifetime:
    - Specifies that the lifetime of the one input parameter gets assigned to the output lifetime

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
```

1. Does not apply

**Example2:**

Apply these rules to figure out the lifetimes of the references in the signature of the `longest` function

```rust
fn longest(x: &str, y: &str) -> &str {
```

1. Compiler applies the first rule:
    - Specifies that each parameter gets its own lifetime. We’ll call them `'a` and `'b`

```rust
WON'T WORK
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

1. Second rule doesn’t apply because there is more than one input lifetime
2. Third rule doesn’t apply because `longest`is a function rather than a method, so none of the parameters are `self`

**Problem:** After using all three rules, Still don’t know what return type’s lifetime is. 

- This is why we got an error trying to compile the code
- Need manual lifetime annotation

## Lifetime Annotations in Method Definitions

When we implement methods on a struct with lifetimes, we use the same syntax as that of generic type parameters

`impl<'a> struct_name<'a> { ...`

Where we declare and use the lifetime parameters depends on whether

- They’re related to the struct fields or
- The method parameters and return values

Lifetime names for struct fields always need to be declared after the `impl` keyword and then used after the struct’s name because those lifetimes are part of the struct’s type

- In method signatures inside the `impl` block, references might be tied to the lifetime of references in the struct’s fields, or they might be independent.
- In addition, the lifetime elision rules often make it so that lifetime annotations aren’t necessary in method signatures

**Example:**

First, we’ll use a method named `level` whose only parameter is a reference to `self` and whose return value is an `i32`, which is not a reference to anything:

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

- Applies first elision rule: Not required to annotate the lifetime of the reference to `self` but lifetime parameter declaration after `impl` and after the type name is required
- 2, 3 don’t apply

********************Example 2:********************

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        return self.part
    }
}

BECOMES IMPLICITLY
 fn announce_and_return_part<'a, 'b>(&'a self, announcement: &'b str) -> &'a str {
        println!("Attention please: {}", announcement);
        return self.part
    }
```

- Applies first elision rule:
    - There are two input lifetimes, so Rust  and gives both `&self` and `announcement` their own lifetimes.
- Applies third  elision rule:
    - One of the parameters is `&self` → the return type gets the lifetime of `&self`, and all lifetimes have been accounted for

## The Static Lifetime

`'static`: Special lifetime which denotes that the affected reference *can* live for the entire duration of the program

All string literals have the `'static` lifetime, which we can annotate as follows:

```rust
let s: &'static str = "I have a static lifetime.";
```

The text of this string is stored directly in the program’s binary, which is always available. 

- Therefore, the lifetime of all string literals is `'static`

Hence, this works:

```rust
fn main() {
    let x;
    {
        let y = "test";
        x = y;
    }
    println!("{}", x); // Y out of scope but print still outputs: "test"
}
```

might see suggestions to use the `'static` lifetime in error messages

- Before specifying `'static` as the lifetime for a reference, think about whether the reference you have actually lives the entire lifetime of your program or not, and whether you want it to