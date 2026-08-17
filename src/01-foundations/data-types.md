# Data Types

Every value has a specific **data type**, which tells the compiler how much memory to allocate and what operations are valid.

Rust is a **statically typed language**, meaning it must know the types of all variables at compile time (though it can infer most of them).

Rust's primitive types are split into two categories:
1. **Scalar types** (a single value).
2. **Compound types** (multiple values grouped together).

## Scalar types

### Integer Types

An integer is a number without a fractional component.

Integers are defined by two properties:

- **Signed (`i`) vs. Unsigned (`u`):** 
    - Signed integers can be positive or negative;
    - unsigned integers can *only* be positive (or zero).
- **Bit Length:** The amount of memory the integer takes up.

| Bit Length | Signed (`i`) | Unsigned (`u`) | Value Range (Signed) | Value Range (Unsigned) |
| --- | --- | --- | --- | --- |
| **8-bit** | `i8` | `u8` | -128 to 127 | 0 to 255 |
| **16-bit** | `i16` | `u16` | -32,768 to 32,767 | 0 to 65,535 |
| **32-bit** | `i32` | `u32` | ≈ -2.14 × 10⁹ to 2.14 × 10⁹ | 0 to ≈ 4.29 × 10⁹ |
| **64-bit** | `i64` | `u64` | ≈ -9.22 × 10¹⁸ to 9.22 × 10¹⁸ | 0 to ≈ 1.84 × 10¹⁹ |
| **128-bit** | `i128` | `u128` | Extremely large | Extremely large |
| **Arch-dependent** | `isize` | `usize` | Depends on target computer architecture (64-bit on 64-bit systems) |  |

> **Default:** If you declare a whole number without specifying a type, Rust defaults to **`i32`**.

```rust,editable
fn main() {
    // Type inference (defaults to i32)
    let default_int = 42;

    // Explicit type annotation
    let small_number: u8 = 255;
    let large_number: i64 = -9_000_000_000; // Underscores `_` can be used as visual separators

    // Integer literals in different bases
    let decimal = 98_222;
    let hex = 0xff;          // 255
    let octal = 0o77;        // 63
    let binary = 0b1111_0000; // 240
    let byte = b'A';         // 65 (u8 only)
}
```

> **Integer Overflow Note:**
> - In **debug builds**, Rust checks for integer overflow and will **panic** at runtime if a value exceeds its type bound (e.g., adding `1` to `255` in a `u8`).
> - In **release builds** (`--release`), Rust wraps around instead (e.g., `255 + 1` becomes `0`).

### Floating-Point Types

Floating-point types represent numbers with decimal points.

Rust has two primitive floating-point types:

* **`f32`**: Single-precision float (32 bits / 4 bytes).
* **`f64`**: Double-precision float (64 bits / 8 bytes). The name "double" simply comes from the fact that it uses twice as many bits as single precision.

> **Default:** Rust defaults to **`f64`** because on modern CPUs it is roughly the same speed as `f32` but offers significantly higher precision.

```rust,editable
fn main() {
    let x = 2.0; // Defaults to f64

    let y: f32 = 3.14159; // Explicit f32

    // Basic arithmetic operations
    let sum = 5.5 + 10.2;         // f64
    let difference = 95.5 - 4.3;  // f64
    let product = 4.0 * 3.0;      // f64
    let quotient = 56.7 / 32.2;   // f64
    let truncated = -5 / 3;       // Integer division evaluates to -1
    print!("{truncated}");
}
```

### The Boolean Type

- Booleans represent a logical truth value: either **`true`** or **`false`**.
- They are exactly **1 byte** in size.
- Type name: `bool`.

```rust,editable
fn main() {
    let t = true;
    let f: bool = false; // With explicit type annotation

    // Common usage in conditionals
    let is_greater = 10 > 5; // Evaluates to true (bool)

    if is_greater {
        println!("10 is greater than 5!");
    }

    // Casting bool -> integer types
    let t_as_u8 = t as u8;       // Yields 1
    let f_as_i32 = f as i32;     // Yields 0

    println!("t as u8: {}", t_as_u8);
    println!("f as i32: {}", f_as_i32);
}
```

### The Character Type

- The `char` type represents a single **Unicode** scalar value.
- It is **4 bytes in size** and can represent far more than ASCII characters.
- **Syntax:** Enclosed with **single quotes** (`'a'`). Double quotes (`"a"`) create a string slice (`&str`).

Because it stores Unicode, `char` natively supports accented letters, Asian scripts, emojis, and zero-width spaces.

```rust,editable
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // Special mathematical symbol
    let heart_eyed_cat = '😻'; // Emojis are valid 4-byte Unicode characters!

    println!("Character: {heart_eyed_cat}");
}
```

### The Unit Type

The unit type `()` actually belongs under **Compound Types** as a zero-element tuple rather than a Scalar type. However, it is an essential primitive in Rust.

- The **unit type** is written as **`()`**.
- It has exactly one value, which is also written as **`()`** (called the unit value).
- It represents an **empty value** or the **absence of a meaningful return value**.
- It occupies **0 bytes of memory**.

In Rust, expressions or functions that don't explicitly return a value implicitly return `()`.

```rust,editable
fn main() {
    // Variable explicitly typed as unit
    let unit: () = ();

    // Functions without an explicit return type return ()
    let result = print_message();
    
    // Check that result is indeed the unit value
    assert_eq!(result, ());
}

fn print_message() {
    println!("Hello!"); // Returns () implicitly
}
```

## Compound Data Types

**Compound types** group multiple values into a single variable.

Rust has two primitive compound types:

### Tuples

- Once declared it is fixed length, cannot grow or shrink in size.
- Each position in the tuple can have a distinct type.

```rust,editable
fn main() {
    // Declaring a tuple with mixed types (f64, u8, char)
    let tup: (f64, u8, char) = (500.0, 1, '🦀');

    // --- Access Pattern 1: Dot Notation (.) ---
    // Access elements directly by their zero-based index
    let five_hundred = tup.0; // f64
    let one = tup.1;          // u8
    let crab = tup.2;         // char

    println!("Dot notation: {}, {}, {}", five_hundred, one, crab);

    // --- Access Pattern 2: Pattern Matching / Destructuring ---
    // Unpack all values from the tuple into individual variables
    let (x, y, z) = tup;
    println!("Destructured: x = {x}, y = {y}, z = {z}");

    // --- Access Pattern 3: Ignoring unwanted fields with `_` ---
    // Extract only what you need while ignoring other positions
    let (_, middle, _) = tup;
    println!("Ignored other fields, extracted middle: {middle}");

    // --- Extra: Mutable Tuples ---
    // You can modify elements in-place if the tuple is declared with `mut`
    let mut mutable_tup = (1, 2);
    mutable_tup.0 = 10;
    println!("Modified tuple: {:?}", mutable_tup);
}
```

### Arrays

- An array groups multiple values where **every element must have the exact same type**.
- Length is fixed at compile-time and is part of the array's type signature.
- Written as `[Type; Length]`. Example: `[i32; 5]` means an array of 5 integers.

> [!IMPORTANT]
> In Rust, arrays have a **fixed length**. They are allocated directly on the **stack** rather than the heap. If you need a collection that can grow or shrink in size, use a **`Vec` (Vector)** instead.

```rust,editable
fn main() {
    // Explicit type annotation: [Type; Count]
    let numbers: [i32; 5] = [1, 2, 3, 4, 5];

    // Array initialization shorthand
    // Creates an array of 5 elements, all set to the value 0
    let zeros = [0; 5]; // Equivalent to: [0, 0, 0, 0, 0]

    // --- Access Pattern 1: Square Bracket Indexing ([index]) ---
    let first = numbers[0];  // 1
    let second = numbers[1]; // 2

    // --- Access Pattern 2: Iteration ---
    for num in numbers {
        println!("{num}");
    }
}
```

#### Out-of-Bounds Memory Safety

When accessing an array element using `array[index]`, Rust enforces **runtime memory safety**.

If you attempt to access an index beyond the end of the array, Rust will compile the program successfully (if the index is dynamic), but will **panic** at runtime rather than allowing invalid memory access (preventing buffer overflow security vulnerabilities).

```rust,editable
fn main() {
    let a = [1, 2, 3, 4, 5];
    let index = 10;

    // 💥 RUNTIME PANIC:
    // 'index out of bounds: the len is 5 but the index is 10'
    let element = a[index]; 
}
```

### Summary Cheat Sheets

| Type Category | Type Name(s) | Default | Size in Memory | Example Syntax |
| --- | --- | --- | --- | --- |
| **Integers** | `i8`–`i128`, `u8`–`u128`, `isize`, `usize` | `i32` | 1 to 16 bytes | `let a: u32 = 10;` |
| **Floats** | `f32`, `f64` | `f64` | 4 or 8 bytes | `let b = 3.14;` |
| **Booleans** | `bool` | N/A | 1 byte | `let c = true;` |
| **Characters** | `char` | N/A | 4 bytes | `let d = '🦀';` |
| **Unit** | `()` | N/A | 0 bytes | `let e: () = ();` |

---

| Feature | Tuple `(T1, T2)` | Array `[T; N]` | Vector `Vec<T>` *(Standard Library)* |
| --- | --- | --- | --- |
| **Multiple Types?** | ✅ Yes | ❌ No (Single type) | ❌ No (Single type) |
| **Fixed Length?** | ✅ Yes | ✅ Yes | ❌ No (Dynamic / Growable) |
| **Memory Allocation** | Stack | Stack | Heap |
| **Access Syntax** | `tuple.0` (dot) | `array[0]` (brackets) | `vec[0]` (brackets) |
