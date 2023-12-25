# Common Collections

Rust’s standard library includes useful data structures called *collections*
Most other data types represent one specific value, but collections can contain multiple values

- Unlike the built-in array and tuple types data these collections point to is stored on the heap
- Means the amount of data does not need to be known at compile time and can grow or shrink as the program runs

Types covered:

- **vector** allows you to store a variable number of values next to each other.
- ***string*** is a collection of characters. We’ve mentioned the `String` type previously, but in this chapter we’ll talk about it in depth.
- **hash map** allows you to associate a value with a particular key. It’s a particular implementation of the more general data structure called a *map*.

[[Vectors  Storing Lists of Values]]

[[Strings  Storing UTF-8 Encoded Text]]

[[Hash Maps  Storing Key Value Pairs]]