# Methods

**Methods** are similar to functions, but with two distinct differences:

1. They are defined inside an **`impl` (implementation) block** attached to a specific struct.
2. Their first parameter is **always `self`**, which represents the instance of the struct the method is being called on.

To add behavior to a struct, you open an `impl` block using the struct’s name:

```rust,editable
struct Rectangle {
    width: u32,
    height: u32,
}

// 1. Open the implementation block for Rectangle
impl Rectangle {
    // 2. Define a method that calculates the area
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    // 3. Call the method using dot notation!
    println!("Area: {} sq pixels", rect1.area());
}
```
> [!NOTE]
> `&self` is shorthand for `self: &Self`.

Methods can borrow or take ownership of `self` in three distinct ways depending on what the method needs to do:

| Parameter | Meaning | Real-World Usage |
| --- | --- | --- |
| **`&self`** | **Immutable Borrow** | Read data from the struct without changing it (Most common!). |
| **`&mut self`** | **Mutable Borrow** | Modify fields inside the struct. |
| **`self`** | **Takes Ownership** | Consumes/destroys the struct (Used when transforming `self` into something else). |

## Example with `&mut self`:

```rust,editable
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Requires &mut self because it alters internal fields!
    fn scale(&mut self, factor: u32) {
        self.width *= factor;
        self.height *= factor;
    }
}

fn main() {
    let mut rect = Rectangle { width: 10, height: 20 };
    rect.scale(2); // Modifies rect in-place!
}
```

## Associated Functions

You can also define functions inside an `impl` block that **do NOT take `self` as a parameter**. These are called **Associated Functions**.

Because they don't operate on a specific instance, they are usually used as **constructors** (functions that build and return a new instance of the struct).

```rust,editable
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Associated Function: Creates a square Rectangle
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}

fn main() {
    // Called using `::` syntax on the Type name, NOT dot notation!
    let sq = Rectangle::square(20);
}
```

> [!NOTE]
> What is `Self`?
> Inside an `impl` block, the keyword `Self` (capital 'S') is an alias for the struct type itself (`Rectangle`).