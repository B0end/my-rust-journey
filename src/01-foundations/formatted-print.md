# Formatted Print

In Rust, outputting text to the console or formatting strings relies on the **`std::fmt`** module. The core macros—`println!`, `print!`, `eprintln!`, and `format!`—use placeholders (`{}`) to format and insert values.

The two primary traits for converting types into readable strings are **`Debug`** (`{:?}`) and **`Display`** (`{}`).

| Feature | `Display` (`{}`) | `Debug` (`{:?}`) |
| --- | --- | --- |
| **Primary Audience** | End-users | Developers |
| **Output Style** | Clean, user-friendly, human-readable | Internal representation (includes struct field names, enum variants) |
| **Derive Macro?** | **No.** Must be manually implemented | **Yes.** Can be derived automatically via `#[derive(Debug)]` |
| **Standard Types** | Implemented for primitives (`i32`, `&str`), but not standard collections (`Vec`, `HashMap`) | Implemented for almost all standard library types |


## Formatted Print Macros

Rust provides four primary macros for outputting formatted text:

* **`println!`**: Prints formatted text to standard output (`stdout`) with a trailing newline.
* **`print!`**: Same as `println!`, but without the trailing newline.
* **`eprintln!`**: Prints to standard error (`stderr`) with a trailing newline (ideal for error messages).
* **`format!`**: Works like `println!`, but returns a formatted `String` instead of printing it to standard output.

```rust,editable
fn main() {
    let name = "Alice";
    let age = 30;

    // 1. println! outputs directly to the screen:
    println!("Hello {}, you are {} years old.", name, age);

    // 2. format! creates a String variable you can save and use later:
    let message: String = format!("Hello {}, you are {} years old.", name, age);

    // Now 'message' holds "Hello Alice, you are 30 years old."
    // You can do whatever you want with it:
    println!("{}", message.to_uppercase());
}
```

#### Positional and Named Parameters

```rust,editable
fn main() {
    // Basic positional placeholder
    println!("Hello, {}!", "world");

    // Positional arguments (0-indexed)
    println!("{0} is from {1}. Yes, {0}!", "Alice", "Wonderland");

    // Named arguments
    println!("{name} lives in {city}", name = "Bob", city = "Madrid");
}
```

## Debug Trait (`{:?}`)

The `Debug` trait prints a representation intended for developers. It is essential for troubleshooting and logging.

* **Placeholder:** `{:?}` (Standard) or `{:#?}` ("Pretty-print" with indentation and newlines).
* **Implementation:** Easily automatically derived using `#[derive(Debug)]`.

```rust,editable
#[derive(Debug)]
struct User {
    username: String,
    age: u32,
}

fn main() {
    let user = User {
        username: String::from("ferris"),
        age: 10,
    };

    // Compact Debug output
    println!("{:?}", user);
    // Output: User { username: "ferris", age: 10 }

    // Pretty-print Debug output (multiline)
    println!("{:#?}", user);
    /* Output:
    User {
        username: "ferris",
        age: 10,
    }
    */
}
```

## Display Trait (`{}`)

The `Display` trait provides clean, user-facing output. Unlike `Debug`, `Display` **cannot be automatically derived** for custom structs or enums; you must manually implement `std::fmt::Display`.

Implementing `Display` automatically provides the `.to_string()` method for your type via `ToString`.

```rust,editable
use std::fmt;

struct Point {
    x: i32,
    y: i32,
}

// Manually implementing fmt::Display for Point
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        // Write formatted output into the buffer `f`
        write!(f, "({}, {}) ✨", self.x, self.y)
    }
}

fn main() {
    let p = Point { x: 5, y: -10 };

    // Display output using `{}`
    println!("The point is at {}", p);
    // Output: The point is at (5, -10)

    // Automatically available via Display implementation
    let point_string = p.to_string();
    assert_eq!(point_string, "(5, -10) ✨");
}
```

>[!NOTE]
> - Use `#[derive(Debug)]` + `{:?}` for quick, effortless developer logs.
> - Implement `Display` + `{}` when you care about custom user-facing formatting.
> - Print individual fields if you only need quick one-off field values.

## Advanced Formatting Specifiers

Within standard `{}` placeholders, you can specify formatting rules for numbers, alignment, and padding:

```rust
fn main() {
    let num = 42;

    println!("Binary: {:b}", num);       // Binary: 101010
    println!("Hex: {:x}", num);          // Hex: 2a
    println!("Octal: {:o}", num);        // Octal: 52

    let pi = 3.14159265;
    println!("2 Decimals: {:.2}", pi);   // 2 Decimals: 3.14

    let val = 5;
    println!("Padded: {:0>5}", val);     // Padded: 00005 (width 5, padded with 0s)
}

```