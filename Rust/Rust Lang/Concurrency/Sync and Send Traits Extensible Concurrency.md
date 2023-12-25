# Sync and Send Traits: Extensible Concurrency

`std::marker` traits `Sync` and `Send` are embedded in the Rust language

## Allowing Transference of Ownership Between Threads with `Send`

`Send` marker trait indicates that ownership of values of the type implementing `Send` can be transferred between threads

Primitive types are `Send`, and types composed entirely of types that are `Send` are also `Send` except:

- `Rc<T>`: cannot implement `Send` because if you cloned an `Rc<T>` value and tried to transfer ownership of the clone to another thread, both threads might update the reference count at the same time
- Raw pointers

## Allowing Access from Multiple Threads with `Sync`

The `Sync` marker trait indicates that it is safe for the type implementing `Sync` to be referenced from multiple threads

Any type `T` is `Sync` if `&T` (an immutable reference to `T`) is `Send`, meaning the reference can be sent safely to another thread

Primitive types are `Sync`, and types composed entirely of types that are `Sync` are also `Sync` except:

- `Rc<T>`:  cannot implement `Sync` because if you cloned an `Rc<T>` value and tried to transfer ownership of the clone to another thread, both threads might update the reference count at the same time
- The `RefCell<T>` type and the family of related `Cell<T>` types are not `Sync`.

## Implementing `Send` and `Sync` Manually Is Unsafe

Because types that are made up of `Send` and `Sync` traits are automatically also `Send` and `Sync`, we don’t have to implement those traits manually

Manually implementing these traits involves implementing unsafe Rust code