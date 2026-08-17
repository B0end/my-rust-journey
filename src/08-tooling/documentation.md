# Documentation

Rust uses specific comment styles to generate standard markdown documentation, automatically test code examples during compilation, and host docs locally or on [docs.rs](https://docs.rs).

* **Outer Doc Comments (`///`):** Document the item that follows them (functions, structs, enums, modules).
* **Inner Doc Comments (`//!`):** Document the containing item (typically placed at the very top of a crate root or module file to describe the module itself).
* **Doc Tests:** Code blocks inside documentation (written inside triple backticks) are automatically extracted and run when you executing `cargo test`. This ensures documentation never becomes outdated.
* **Standard Sections:** Idiomatic Rust documentation uses specific Markdown headers (`# Examples`, `# Errors`, `# Panics`, `# Safety`).

Testing your Rust documentation code locally involves running Cargo's built-in doc testing framework and opening the generated HTML documentation in your browser to inspect its formatting.

1. **Set Up a Library Project:** Prerequisite.
Doc-tests are **only compiled and executed for library crates**, not binary (`bin`) crates. Create a new library project:

```bash
cargo new --lib math_utils
cd math_utils
```

2. **Add the Documented Code:** src/lib.rs.
Replace the contents of `src/lib.rs` with the fully documented code:

```rust
//! # Math Utilities Crate
//!
//! `math_utils` provides simple arithmetic operations with robust error handling.

/// Represents errors that can occur during mathematical operations.
#[derive(Debug, PartialEq, Eq)]
pub enum MathError {
    /// Attempted to divide a number by zero.
    DivisionByZero,
}

/// Divides a numerator by a denominator.
///
/// # Examples
///
/// ```
/// use math_utils::{divide, MathError};
///
/// let result = divide(6.0, 3.0);
/// assert_eq!(result, Ok(2.0));
/// ```
///
/// ```
/// use math_utils::{divide, MathError};
///
/// let result = divide(5.0, 0.0);
/// assert_eq!(result, Err(MathError::DivisionByZero));
/// ```
pub fn divide(numerator: f64, denominator: f64) -> Result<f64, MathError> {
    if denominator == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(numerator / denominator)
    }
}

/// Calculates the square root of a non-negative number.
///
/// # Panics
///
/// Panics if `value` is negative.
///
/// # Examples
///
/// ```
/// use math_utils::strict_sqrt;
///
/// assert_eq!(strict_sqrt(9.0), 3.0);
/// ```
///
/// ```should_panic
/// use math_utils::strict_sqrt;
///
/// strict_sqrt(-4.0);
/// ```
pub fn strict_sqrt(value: f64) -> f64 {
    if value < 0.0 {
        panic!("Cannot compute square root of a negative number: {}", value);
    }
    value.sqrt()
}
```

3. **Run Doc Tests:**
Execute `cargo test`. Cargo automatically extracts code blocks inside `///` comments, compiles them into individual binaries, and runs them:

```bash
cargo test
```

4. **Target Doc Tests Specifically:**
If you have unit tests or integration tests and want to run **only** the documentation tests:

```bash
cargo test --doc
```

5. **Build and Render HTML Documentation:**
Compile the doc comments into HTML pages and open them directly in your default web browser:

```bash
cargo doc --open
```

This generates HTML files inside the `target/doc/math_utils/index.html` directory and displays rendering, clickable links, and styled code blocks.