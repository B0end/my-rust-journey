# Generic Data Types

Imagine you want a function that finds the largest number in a list of `i32` integers:

```rust
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}
```

What if you now need the exact same logic for a list of `char` values or `f64` floats? Without generics, you would have to copy and paste the entire function just to change the type annotations!

> [!NOTE]
> By convention, Rust uses short, single uppercase letters for generic type parameters (most commonly `T` for "Type").

## Generic Functions

To make a function generic, declare the generic type name inside angle brackets `<T>` **between the function name and the parameter list**:

```rust
// Read as: "Function `largest` is generic over type `T`"
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        // (Note: We'll see in 4.2 why `>` requires a Trait bound!)
        if item > largest {
            largest = item;
        }
    }
    largest
}
```

Now, `largest` can accept a slice of *any* type `T` (`&[i32]`, `&[char]`, `&[String]`, etc.)!

## Generic Structs

You can also define structs that hold generic field values using `<T>` syntax:

### A. Single Generic Type (`T`)

Both `x` and `y` **must be of the exact same type**:

```rust,editable
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer_point = Point { x: 5, y: 10 };       // Point<i32>
    let float_point   = Point { x: 1.0, y: 4.0 };    // Point<f64>

    // ❌ COMPILE ERROR! `x` and `y` must be the same type T!
    // let wont_work = Point { x: 5, y: 4.0 };
}
```

### B. Multiple Generic Types (`T`, `U`)

If you want fields to potentially have **different types**, declare multiple generic parameters like `<T, U>`:

```rust,editable
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let mixed_point = Point { x: 5, y: 4.0 }; // Point<i32, f64> - Works!
}
```

## Generic Enums

Both `Option<T>` and `Result<T, E>` in the standard library are generic enums:

```rust
// Holds Some value of type `T`, or None
enum Option<T> {
    Some(T),
    None,
}

// Holds Ok value of type `T`, or Err value of type `E`
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

This flexibility is why `Option` can hold an `Option<i32>`, `Option<String>`, or `Option<User>` without needing separate custom enums for each!

## Generic Methods (`impl<T>`)

```rust,editable
struct Point<T> {
    x: T,
    y: T,
}

// 1. Generic implementation for ANY type T
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

// 2. Concrete implementation ONLY for Point<f64>!
impl Point<f64> {
    fn distance_from_origin(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10 };
    let p2 = Point { x: 1.0, y: 2.0 };

    println!("p1.x = {}", p1.x());
    
    // `distance_from_origin()` ONLY exists on Point<f64>!
    println!("p2 distance = {}", p2.distance_from_origin());
}
```

## Performance of Generics: Monomorphization

Does using generic types slow down your program at runtime? **No! Rust generics have ZERO runtime cost.**

The Rust compiler uses a process called **Monomorphization** at compile time:

1. It looks at all the concrete types your code actually uses with a generic function or struct.
2. It generates specific, duplicated machine code for each concrete type!

If you write:

```rust
let integer = Point { x: 5, y: 10 };
let float   = Point { x: 1.0, y: 4.0 };
```

During compilation, Rust turns `Point<T>` into two distinct structs: `Point_i32` and `Point_f64`.

Because the conversion happens during compilation, **your code runs at the exact same blazing-fast speed as if you had written separate structs manually!**

## Summary Cheat Sheet

| Feature | Syntax | Meaning |
| --- | --- | --- |
| **Generic Function** | `fn foo<T>(arg: T)` | Function working over type `T` |
| **Single Type Struct** | `struct Point<T> { x: T, y: T }` | Both fields must be same type `T` |
| **Multi-Type Struct** | `struct Point<T, U> { x: T, y: U }` | Fields can be different types |
| **Generic Methods** | `impl<T> Point<T> { ... }` | Methods available on any `Point<T>` |
| **Concrete Methods** | `impl Point<f64> { ... }` | Methods *only* available on `Point<f64>` |
