# Characteristics of OOP Languages

Object-oriented programs are made up of objects

- An *object* packages both data and the procedures that operate on that data
- The procedures are typically called *methods* or *operations*

Rust is object-oriented: structs and enums have data, and `impl` blocks provide methods on structs and enums

## Encapsulation that Hides Implementation Details

**encapsulation:** implementation details of an object aren’t accessible to code using that object

- Therefore, the only way to interact with an object is through its public API
- Code using the object shouldn’t be able to reach into the object’s internals and change data or behavior directly
- Enables the programmer to change and refactor an object’s internals without needing to change the code that uses the object

Rust uses the `pub` keyword to decide which modules, types, functions, and methods in our code should be public

- By default everything else is private

**Example:** define a struct `AveragedCollection` that has a field containing a vector of `i32` values and field that contains the average of the values in the vector

- Average doesn’t have to be computed on demand whenever anyone needs it

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
```

The struct is marked `pub` so that other code can use it, but the fields within the struct remain private

- Important because we want to ensure that whenever a value is added or removed from the list, the average is also updated

```rust
impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            }
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

The public methods `add`, `remove`, and `average` are the only ways to access or modify data in an instance of `AveragedCollection`

- When modifying the array, the average is recomputed: `update_average`

## Inheritance as a Type System and as Code Sharing

**Inheritance:** A mechanism whereby an object can inherit elements from another object’s definition

- Thus gaining the parent object’s data and behavior without you having to define them again

In Rust: There is no way to define a struct that inherits the parent struct’s fields and method implementations (without using a macro)

You can however use default trait method implementations

- Write method once in trait, don’t have to rewrite later
- Can also override the default implementation in child

The other reason to use inheritance relates to the type system: to enable a child type to be used in the same places as the parent type: ************************polymorphism************************

### Polymorphism

polymorphism: means that you can substitute multiple objects for each other at runtime if they share certain characteristics

**Note:** May think polymorphism is synonymous with inheritance

- actually a more general concept that refers to code that can work with data of multiple types
    - For inheritance, those types are generally subclasses
- ***bounded parametric polymorphism:*** Rust instead uses generics to abstract over different possible types and trait bounds to impose constraints on what those types must provide

Regular polymorphism has fallen out of style