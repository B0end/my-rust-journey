# Closures and Iterators

## Closures (Anonymous Functions That Capture Their Environment)

A **Closure** is an anonymous function you can save in a variable or pass as an argument to other functions. Unlike standard functions (`fn`), closures can **capture values from the scope in which they are defined**.

Closures use vertical pipes `|` for parameters instead of parentheses `()`:

```rust,editable
fn main() {
    // Standard function
    fn add_one_v1(x: u32) -> u32 { x + 1 }

    // Closure with explicit type annotations
    let add_one_v2 = |x: u32| -> u32 { x + 1 };

    // Closure with type inference & expression body (Most common!)
    let add_one_v3 = |x| x + 1;
}
```

Closures can capture variables from their outer scope in three ways, mapping directly to ownership rules:

1. **Borrowing Immutably (`Fn`)**: Reads values from the environment.
2. **Borrowing Mutably (`FnMut`)**: Modifies values in the environment.
3. **Taking Ownership (`FnOnce`)**: Moves values out of the environment.

Example: Immutable Borrow

```rust,editable
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");

    // Captures `list` by immutable reference (&list)
    let only_borrows = || println!("From closure: {list:?}");

    only_borrows();
    println!("After calling closure: {list:?}");
}
```

If you need a closure to take full ownership of the variables it uses (e.g., when passing data to a new thread), use the **`move`** keyword:

```rust,editable
use std::thread;

fn main() {
    let list = vec![1, 2, 3];

    // `move` forces ownership of `list` into the closure/thread
    thread::spawn(move || println!("From thread: {list:?}"))
        .join()
        .unwrap();

    // ❌ COMPILE ERROR! `list` was moved into the thread!
    // println!("{list:?}"); 
}
```

## Iterators (Processing a Series of Items)

An **Iterator** is responsible for the logic of stepping through a sequence of items one by one.

In Rust, iterators are **lazy**: they have no effect until you call methods that consume the iterator to use it up! 

All iterators implement a standard library trait named `Iterator`:

```rust
pub trait Iterator {
    type Item; // The type of element being iterated over

    // Advances the iterator and returns the next value
    fn next(&mut self) -> Option<Self::Item>; 
}
```

Calling `.next()` manually advances the iterator:

```rust,editable
fn main() {
    let v1 = vec![1, 2];

    let mut v1_iter = v1.iter(); // Must be `mut` because .next() changes internal state!

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), None); // Sequence ended!
}
```

### Three Ways to Produce Iterators

Depending on how you want to handle ownership of the elements:

| Method | What it yields | Ownership |
| --- | --- | --- |
| **`v.iter()`** | `&T` | Borrows elements immutably |
| **`v.iter_mut()`** | `&mut T` | Borrows elements mutably |
| **`v.into_iter()`** | `T` | **Takes ownership** of collection |

### Consuming Adaptors

Methods that call `.next()` internally and exhaust the iterator are called **Consuming Adaptors**.

Example: `.sum()`

```rust,editable
fn main() {
    let v1 = vec![1, 2, 3];
    let total: i32 = v1.iter().sum(); // Consumes v1_iter!
    println!("Total: {total}"); // 6
}
```

### Iterator Adaptors

**Iterator Adaptors** are methods that transform an iterator into a *different* iterator. Because iterators are lazy, you must chain a consuming adaptor (like `.collect()`) at the end to actually perform the work!

- `.map()` — Transform elements

Applies a closure to each element:

```rust,editable
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3];

    // Multiply every item by 2 and collect into a new Vector
    let v2: Vec<i32> = v1.iter().map(|x| x * 2).collect();

    println!("{v2:?}"); // [2, 4, 6]
}
```

- `.filter()` — Keep elements that match a condition

Uses a closure returning a boolean to filter elements:

```rust,editable
fn main() {
    let v1: Vec<i32> = vec![1, 2, 3, 4, 5, 6];

    // Keep only even numbers
    let evens: Vec<i32> = v1.into_iter().filter(|x| x % 2 == 0).collect();

    println!("{evens:?}"); // [2, 4, 6]
}
```

### Performance: Iterators vs. Loops

You might wonder: *Does chaining `.map()`, `.filter()`, and closures make Rust code slower than writing manual `for` loops?*

**Answer: No! Iterators are Zero-Cost Abstractions.**

The Rust compiler monomorphizes and aggressively optimizes iterator chains into raw machine code—often compiling down to the **exact same assembly code** as (or even faster than) a manual loop because the compiler can safely eliminate array bounds checks!

## Summary Cheat Sheet

| Feature | Syntax / Code | Description |
| --- | --- | --- |
| **Closure** | `\|x\| x + 1` | Anonymous function capturing scope |
| **Move Closure** | `move \|\| println!("{x}")` | Takes ownership of environment variables |
| **Immutable Iter** | `v.iter()` | Yields references `&T` |
| **Owned Iter** | `v.into_iter()` | Takes ownership, yields values `T` |
| **Transform** | `iter.map(\|x\| ...)` | Applies closure to each item |
| **Filter** | `iter.filter(\|x\| ...)` | Keeps items where closure returns `true` |
| **Collect** | `iter.collect()` | Consumes iterator into a collection (e.g. `Vec`) |
