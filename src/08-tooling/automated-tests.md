# Automated Tests

At its simplest, a test in Rust is just a function annotated with the **`#[test]`** attribute.

When you run **`cargo test`**, Rust compiles a test runner binary that executes all functions annotated with `#[test]` and reports whether they passed or failed.

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```

If a test function runs to completion without crashing (panicking), the test **passes**. If the function panics, the test **fails**!

Rust provides three main standard macros to test assertions:

- `assert!(expression)`: Checks that a boolean expression evaluates to `true`. If `false`, it panics and fails the test.

```rust
#[test]
fn Test_is_larger() {
    let larger = 10 > 5;
    assert!(larger);
}
```

- `assert_eq!(left, right)` & `assert_ne!(left, right)`
    * `assert_eq!` tests that two values are **equal** (`left == right`).
    * `assert_ne!` tests that two values are **not equal** (`left != right`).

```rust
#[test]
fn test_add() {
    assert_eq!(2 + 2, 4);
    assert_ne!(2 + 2, 5);
}
```

You can pass custom formatted failure messages as extra arguments to any assertion macro:

```rust
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{result}`"
    );
}
```

## Testing for Expected Panics with `#[should_panic]`

Sometimes you want to test that code **fails correctly** when given invalid inputs (e.g., passing a negative number where prohibited).

Annotate the test with **`#[should_panic]`**:

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {value}.");
        }
        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "between 1 and 100")] // Verifies the panic message contains this substring!
    fn greater_than_100_panics() {
        Guess::new(200); // Trigger panic intentionally
    }
}
```

## Using `Result<T, E>` in Tests

Instead of panicking on failure, test functions can also return a **`Result<T, E>`**. This allows you to use the `?` operator inside tests!

```rust
#[test]
fn it_works() -> Result<(), String> {
    let result = 2 + 2;

    if result == 4 {
        Ok(())
    } else {
        Err(String::from("two plus two does not equal four"))
    }
}
```

## Controlling How Tests Run with `cargo test`

You can pass command-line arguments to `cargo test` to customize execution:

By default, tests run in parallel using multiple threads. To run tests one at a time (e.g., if tests share a database or file state):

```bash
cargo test -- --test-threads=1
```

By default, Rust captures standard output (`println!`) for tests that pass. To see print outputs regardless of pass/fail status:

```bash
cargo test -- --show-output
```

Run only tests that match a specific name substring:

```bash
cargo test add // Runs `test_add`, `add_one`, etc.
```

Mark long-running tests with **`#[ignore]`**:

```rust
#[test]
#[ignore]
fn expensive_test() {
    // Code that takes 5 minutes to run...
}
```

Run ignored tests explicitly with:

```bash
cargo test -- --ignored
```

## Unit Tests vs. Integration Tests

Rust organizes testing into two main categories:

### A. Unit Tests

* **Where:** Written in the **same file** as the code being tested inside a `tests` module marked with `#[cfg(test)]`.
* **Purpose:** Test small, isolated units of code (like a single function).
* **Privacy:** Can test **private** functions!

```rust
// src/lib.rs

pub fn add_two(a: i32) -> i32 {
    internal_helper(a)
}

fn internal_helper(a: i32) -> i32 { // Private function!
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_internal_helper() {
        assert_eq!(internal_helper(2), 4); // ✅ Unit tests can call private functions!
    }
}
```

> **What does `#[cfg(test)]` do?**
> It tells Cargo to **only** compile this module when you run `cargo test`, keeping test code out of your production binary build!

### B. Integration Tests

* **Where:** Written in a separate **`tests/`** directory at the root level (next to `src/`).
* **Purpose:** Test whether multiple parts of your library work together correctly from the perspective of an external caller.
* **Privacy:** Can **ONLY** test the public API of library crates (`pub`).

Project Directory Tree:

```text
my_project/
├── Cargo.toml
├── src/
│   └── lib.rs
└── tests/
    └── integration_test.rs
```

```rust
// tests/integration_test.rs
use my_project::add_two; // Import library crate as an external dependency

#[test]
fn it_adds_two() {
    assert_eq!(add_two(2), 4);
}
```

Each `.rs` file inside the `tests/` directory is compiled as a separate individual crate!


## Summary Cheat Sheet

| Feature / Command | Syntax | What it does |
| --- | --- | --- |
| **Mark Test Function** | `#[test]` | Registers function with test runner |
| **Assert Equal** | `assert_eq!(a, b);` | Asserts `a == b` |
| **Assert True** | `assert!(expression);` | Asserts condition is `true` |
| **Test Expected Panic** | `#[should_panic]` | Pass if code panics |
| **Ignore Test** | `#[ignore]` | Skip test during standard `cargo test` |
| **Run Specific Test** | `cargo test test_name` | Filters tests by name substring |
| **Run Ignored Tests** | `cargo test -- --ignored` | Executes ignored tests |
| **Conditional Build** | `#[cfg(test)]` | Compiles block only during tests |
