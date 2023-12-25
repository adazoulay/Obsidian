# Raw Pointers: Common Methods

# Unsafe Rust and Raw Pointer Operations

## Raw Pointers

**`is_null(self)`**: Determines if the pointer is null.

- Params: self
- Returns: bool
- Example:
    
    ```rust
    let ptr: *const i32 = null();
    assert!(unsafe { ptr.is_null() });
    ```
    

**`offset(self, count: isize)`**: Calculates the offset from a pointer.

- Params: self, count (isize)
- Returns: Raw pointer
- Safety: The result may not be a valid pointer. The caller must ensure that the computed pointer remains within the same allocated object.
- Example:
    
    ```rust
    let a = [1, 2, 3, 4];
    let ptr = a.as_ptr();
    unsafe {
        assert_eq!(*ptr.offset(2), 3);
    }
    ```
    

**`add(self, count: usize)`**: Calculates the offset from a pointer in bytes.

- Params: count (usize)
- Returns: Raw pointer
- Safety: The computed pointer must either be null or all of the following conditions must be met:
    - it must be properly aligned,
    - it must be "in bounds",
    - the offset must not overflow an isize,
    - the new pointer must be in the same allocated object.
- Example:
    
    ```rust
    unsafe {
        let new_ptr = source_ptr.add(2);
    }
    ```
    

**`read(self)`**: Reads the value from the pointer's memory address.

- Params: self
- Returns: Value read from the pointer
- Safety: Behavior is undefined if any of the following conditions are violated:
    - `src` must be valid for reads.
    - `src` must be properly aligned.
- Example:
    
    ```rust
    let x = 12;
    let ptr = &x as *const i32;
    let val = unsafe { ptr.read() };
    ```
    

**`write(self, val)`**: Writes a value to the pointer's memory address.

- Params: self, value to write
- Returns: None
- Safety: Behavior is undefined if any of the following conditions are violated:
    - `dst` must be valid for writes.
    - `dst` must be properly aligned.
- Example:
    
    ```rust
    let mut x = 7;
    let p = &mut x as *mut i32;
    unsafe { p.write(12) };
    ```
    

**`read_unaligned(self)`**: Performs a volatile read of the value from the pointer's memory address, bypassing alignment.

- Params: self
- Returns: Value read from the pointer
- Safety: Behavior is undefined if any of the following conditions are violated:
    - `src` must be valid for reads.
- Example:
    
    ```rust
    let x = 12;
    let ptr = &x as *const i32;
    let val = unsafe { ptr.read_unaligned() };
    ```
    

**`write_unaligned(self, val)`**: Performs a volatile write to the pointer's memory address, bypassing alignment.

- Params: self, value to write
- Returns: None
- Safety: Behavior is undefined if any of the following conditions are violated:
    - `dst` must be valid for writes.
- Example:
    
    ```rust
    let mut x = 7;
    let p = &mut x as *mut i32;
    unsafe { p.write_unaligned(12) };
    ```
    

**`copy_to(self, dst: *mut T, count: usize)`**: Copies `count * size_of::<T>()` bytes from `self` to `dst`.

- Params: self, destination pointer, count of elements to copy
- Returns: None
- Safety: The region of memory the source pointer covers must be valid for reads of `count * size_of::<T>()` bytes.
- Example:
    
    ```rust
    let a = [1, 2, 3, 4];
    let mut b = [0; 4];
    let a_ptr = a.as_ptr();
    let b_ptr = b.as_mut_ptr();
    unsafe {
        a_ptr.copy_to(b_ptr, 4);
    }
    assert_eq!(a, b);
    ```
    

**`copy_from(self, src: *const T, count: usize)`**: Copies `count * size_of::<T>()` bytes from `src` to `self`.

- Params: self, source pointer, count of elements to copy
- Returns: None
- Safety: The region of memory the destination pointer covers must be valid for writes of `count * size_of::<T>()` bytes.
- Example:
    
    ```rust
    let a = [1, 2, 3, 4];
    let mut b = [0; 4];
    let a_ptr = a.as_ptr();
    let b_ptr = b.as_mut_ptr();
    unsafe {
        b_ptr.copy_from(a_ptr, 4);
    }
    assert_eq!(a, b);
    ```