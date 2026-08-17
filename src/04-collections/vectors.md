# Vectors (`Vec<T>`)

A **Vector** allows you to store more than one value in a single data structure, putting all the values next to each other in memory (contiguous memory layout).

> [!IMPORTANT]
> A vector can only store values of the **same type** (`T`).

## Create vector

There are two main ways to create a vector:

1. Using `Vec::new()`

```rust,editable
// Creates an empty vector that will hold 32-bit integers
let v: Vec<i32> = Vec::new();
```

2. Using the `vec!` Macro (Most Common!)
    - If you want to initialize a vector with starting values, use the `vec!` macro. Rust will automatically infer the type:

```rust,editable
// Rust infers that `v` is of type Vec<i32>
let v = vec![1, 2, 3];
```

## Add elements to a vector

To add elements to a vector, mark it as **`mut`** and use the **`.push()`** method:

```rust,editable
fn main() {
    let mut v = Vec::new();

    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
} // 👈 `v` goes out of scope here and is freed from Heap RAM!
```

## Read a value from a vector

There are two main ways to read a value from a vector: using **Indexing** (`[]`) or using the **`.get()`** method.

```rust,editable
fn main() {
    let v = vec![10, 20, 30, 40, 50];

    // Method 1: Direct Indexing (Panics if out-of-bounds!)
    let third: &i32 = &v[2];
    println!("The third element is {third}");

    // Method 2: The .get() method (Returns Option<&T> - Safe!)
    let third_option: Option<&i32> = v.get(2);
    
    match third_option {
        Some(element) => println!("The third element is {element}"),
        None => println!("There is no third element."),
    }
}
```

| Access Method | What happens on valid index? | What happens if index is OUT OF BOUNDS? |
| --- | --- | --- |
| **`&v[index]`** | Returns `&T` | **Program Crashes immediately (Panics)!** |
| **`v.get(index)`** | Returns `Some(&T)` | **Returns `None` safely without crashing.** |

> [!TIP]
> Use `[]` when you are 100% sure the index exists. Use `.get()` when accepting user input or when the index might be out of bounds.

## Borrowing Rules

Because vectors store their elements contiguously in heap memory, adding a new element might require allocating a larger block of memory and copying all old elements to the new space.

This means **holding an immutable reference to an element while modifying the vector is forbidden by the compiler!**

```rust,editable
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0]; // 🤓 Immutable borrow of first element!

    v.push(6); // ❌ COMPILE ERROR! Cannot borrow `v` as mutable because `first` is still borrowed!

    println!("The first element is: {first}");
}
```

## Iterating Over Values in a Vector

To access each element in a vector in turn, iterate over it using a `for` loop:

### Reading values (Immutable Iteration):

```rust,editable
fn main() {
    let v = vec![100, 32, 57];

    for i in &v {
        println!("{i}");
    }
}
```

### Modifying values in-place (Mutable Iteration):

To change the elements inside a vector during iteration, iterate over a mutable reference `&mut v` and use the dereference operator (`*`) to modify the value:

```rust,editable
fn main() {
    let mut v = vec![100, 32, 57];

    for i in &mut v {
        *i += 50; // Use `*` (dereference) to modify the actual value on heap!
    }

    // `v` is now [150, 82, 107]
}
```

## Storing Multiple Types with Enums

What if you *really* need a vector to store different types of data (e.g., integers, floats, and strings)?

You can define an **Enum** whose variants hold those different data types, and then create a vector of that Enum type!

```rust,editable
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

fn main() {
    // A single vector holding different types wrapped in an Enum!
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
}
```

## Summary Cheat Sheet

| Action | Syntax |
| --- | --- |
| **Create empty** | `let mut v = Vec::new();` |
| **Create with values** | `let v = vec![1, 2, 3];` |
| **Add element** | `v.push(4);` |
| **Read (Unsafe)** | `let item = &v[0];` |
| **Read (Safe)** | `let item = v.get(0);` |
| **Iterate (Read)** | `for i in &v { ... }` |
| **Iterate (Modify)** | `for i in &mut v { *i += 1; }` |

