# Smart Pointers

[[Using Box T  to Point to Heap Data]]

[[Deref Trait  Treating Smart Pointers Like Regular References]]

[[Drop Trait  Running Code on Cleanup]]

[[Rc T   Reference Counted Smart Pointer]]

[[RefCell T   Interior Mutability Pattern]]

[[Reference Cycles Can Leak Memory]]

### Pointers

General concept for a variable that contains an address in memory

- address refers to, or “points at,” some other data

Most Common kind of pointer in Rust is a reference `&`

- Borrow the value they point to
- Don’t have any special capabilities other than referring to data, no overhead

### **Smart pointers**

Act like a pointer but also have additional metadata and capabilities

- In many cases, smart pointers *own* the data they point to
- `String` and `Vec<T>` are both smart pointers because they own some memory and allow you to manipulate it
    - They also have metadata and extra capabilities or guarantees
    - Ex: `String` stores its capacity as metadata and ensure its data will always be valid UTF8

Smart pointers are usually implemented using structs

Unlike an ordinary struct, smart pointers implement the `Deref` and `Drop` traits.

- `Deref` trait allows an instance of the smart pointer struct to behave like a reference
    - Can write your code to work with either references or smart pointers.
- `Drop` trait allows you to customize the code that’s run when an instance of the smart pointer goes out of scope.

Pointers in the standard library:

- `Box<T>` for allocating values on the heap
- `Rc<T>`, a reference counting type that enables multiple ownership
- `Ref<T>` and `RefMut<T>`, accessed through `RefCell<T>`, a type that enforces the borrowing rules at runtime instead of compile time

Reasons to choose `Box<T>`, `Rc<T>`, or `RefCell<T>`:

- `Rc<T>` enables multiple owners of the same data; `Box<T>` and `RefCell<T>` have single owners.
- `Box<T>` allows immutable or mutable borrows checked at compile time; `Rc<T>` allows only immutable borrows checked at compile time; `RefCell<T>` allows immutable or mutable borrows checked at runtime.
- Because `RefCell<T>` allows mutable borrows checked at runtime, you can mutate the value inside the `RefCell<T>` even when the `RefCell<T>` is immutable.

[[Common Smart Pointers]]