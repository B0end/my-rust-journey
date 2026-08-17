# Packages, Crates, and Modules

As your Rust programs grow from small scripts into real-world applications, putting everything in a single `main.rs` file becomes impossible to manage. Rust provides a full system for organizing, structuring, and reusing code safely.

Rust's module system consists of four main building blocks, organized from largest to smallest:

1. **Packages:** A Cargo feature that lets you build, test, and share crates. (Contains a `Cargo.toml`).
2. **Crates:** A tree of modules that produces a library or an executable.
3. **Modules & `use`:** Let you organize scope, privacy, and organization of paths.
4. **Paths:** A way of naming an item (such as a struct, function, or module).

## Crates

A **Crate** is the smallest amount of code that the Rust compiler considers at one time.

Crates come in two forms:

* **Binary Crate:** An executable program you can run (has a `main()` function).
* **Library Crate:** Code meant to be shared with other programs (has no `main()` function; defines reusable helper functions, structs, etc.).

## Packages

A **Package** is a bundle of one or more crates that provides a set of functionality. It is defined by a **`Cargo.toml`** file that describes how to build those crates.

> [!NOTE]
> **Cargo Rules for Packages:**
> * A package **MUST** contain at least one crate (either library or binary).
> * It can contain **at most ONE library crate**.
> * It can contain **any number of binary crates**.

> [!NOTE]
> **Cargo Conventions:**
> * `src/main.rs`: Cargo knows this is the crate root of a **binary crate** with the same name as the package.
> * `src/lib.rs`: Cargo knows this is the crate root of a **library crate** with the same name as the package.

## Defining Modules to Control Scope and Privacy

Modules allow you to organize code within a crate for readability and reuse. They also control **privacy** (whether code can be accessed from outside the module).

Let's build a restaurant front-of-house system:

```rust
// src/lib.rs

mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}
```

Modules can hold definitions for functions, structs, enums, constants, or even other nested modules!

## Paths for Referring to an Item in the Module Tree

To call a function inside a module, you need to use a **Path** (just like navigating a file system on your OS):

* **Absolute Path:** Starts from the crate root (using `crate::`).
* **Relative Path:** Starts from the current module (using `self::`, `super::`, or an identifier in the current module).

```rust
mod front_of_house {
    pub mod hosting { // `pub` makes this module public!
        pub fn add_to_waitlist() {} // `pub` makes this function public!
    }
}

pub fn eat_at_restaurant() {
    // 1. Absolute path (Starts from `crate` root)
    crate::front_of_house::hosting::add_to_waitlist();

    // 2. Relative path (Starts from current scope)
    front_of_house::hosting::add_to_waitlist();
}
```

## The Privacy Boundary (`pub` Keyword)

By default in Rust, **EVERYTHING IS PRIVATE**!

* Parent modules **cannot** see private items inside their child modules.
* Child modules **can** see everything inside their parent modules (ancestor scope).

To make an item visible to parent modules, you must mark it with the **`pub`** keyword:

```rust
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,      // Public field
        seasonal_fruit: String, // PRIVATE field!
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");

    // We can read and modify `toast` because it is pub:
    meal.toast = String::from("Wheat");

    // ❌ COMPILE ERROR! `seasonal_fruit` is private!
    // meal.seasonal_fruit = String::from("blueberries");
}   
```

> [!IMPORTANT]
> **Struct vs Enum Privacy:**
> * **Structs:** Marking a struct as `pub` does **NOT** make its fields public automatically. Each field must be individually marked `pub`.
> * **Enums:** If you mark an enum as `pub`, **ALL of its variants automatically become public!**

## Bringing Paths into Scope with `use`

Typing out full paths like `crate::front_of_house::hosting::add_to_waitlist()` every time gets tedious.

The **`use`** keyword brings a path into the current scope once, so you can call it with a shorter name (similar to `import` in Python/JavaScript or `using` in C++):

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

// Bring `hosting` module into scope
use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist(); // Much cleaner!
}
```

## Shortcuts for `use` Syntax

If bringing two items with the same name into scope causes a conflict, rename one with `as`:

```rust
use std::fmt::Result;
use std::io::Result as IoResult; // Renamed to avoid collision!
```

Combine multiple imports from the same parent using `{}`:

```rust
// Instead of this:
// use std::cmp::Ordering;
// use std::io;

// Write this:
use std::{cmp::Ordering, io};
```

Brings **ALL** public items from a module into scope (use sparingly!):

```rust
use std::collections::*;
```

## Splitting Modules into Different Files

In real applications, you don't keep all modules in one file. You split them into separate files following Rust's folder naming rules.

Suppose we want to move `front_of_house` into its own file:

File Tree:

```text
src/
├── lib.rs
└── front_of_house.rs
```

`src/front_of_house.rs`

```rust
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

`src/lib.rs`

```rust
// Declares that the `front_of_house` module exists in `src/front_of_house.rs`
mod front_of_house; 

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

## Summary Cheat Sheet

| Keyword | Purpose |
| --- | --- |
| **`mod`** | Defines a new module |
| **`pub`** | Makes a module, struct field, or function public |
| **`use`** | Brings a path into scope for shorter typing |
| **`as`** | Renames an imported item in scope |
| **`super::`** | Refers to the parent module (like `..` in paths) |
| **`crate::`** | Starts an absolute path from the crate root |
