# Introduction

Rust provides C/C++-level performance and low-level memory control, but guarantees **memory safety and thread safety at compile time** without using a garbage collector.

Most languages handle memory in one of two ways:

1. **Garbage Collection (Python, Java, Go):** Easy to write, but introduces runtime overhead and pause times.
2. **Manual Management (C, C++):** Fast and precise, but prone to crashes, buffer overflows, and memory leaks.

Rust introduces a third way: **Ownership** ✨.

---

## Installation

The official toolchain installer, **`rustup`**, manages your Rust compiler (`rustc`), package manager (`cargo`), and standard library targets in one place.

### Linux / macOS

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Windows

You have two primary setup choices depending on your C/C++ build tools environment:

1. **MSVC Toolchain (Recommended for general Windows development):**
* Download and run [rustup-init.exe](https://rustup.rs/).
* Ensure you have **Visual Studio C++ Build Tools** installed (the installer will warn you and provide a link if you don't).

2. **GNU Toolchain / MinGW (Alternative if you use MinGW or Git Bash):**
* During the `rustup` setup, you can customize the default host triple to `x86_64-pc-windows-gnu`.

Confirm all core tools were installed correctly by checking their versions:

```bash
rustc --version   # The Rust compiler
cargo --version   # The build tool and package manager
rustup --version  # The toolchain installer/updater
```

### IDE & Editor Integration

To get autocompletion, inline errors, and go-to-definition, install the **`rust-analyzer`** extension in your editor:

* **VS Code:** Install the official `rust-analyzer` extension (and disable the legacy `Rust` extension if installed).
* **Neovim / Vim:** Use standard LSP clients (`nvim-lspconfig`) configured with `rust-analyzer`.
* **JetBrains (CLion / RustRover):** Install the native Rust plugin or use RustRover directly.

---

## Hello, World!

```rust,editable
fn main() {
    println!("Hello, world!");
}
```

### `fn main() {`

* **`fn`**: The keyword used to declare a function.
* **`main`**: The entry point of every Rust binary executable program. When your program starts, execution begins here.
* **`()`**: Defines the function parameters. Unlike languages like C/C++ or Java—where `main` can accept parameters like `int main(int argc, char *argv[])` or `public static void main(String[] args)`—Rust's main signature is fixed, zero parameters allowed!
    * **How do you get Command-Line Arguments in Rust?** Instead of passing arguments into `main()`, you pull them from the standard library inside the function body using `std::env::args`.
* **`{`**: Opens the body of the function. All code logic must sit inside curly brackets `{}`.

`{}` define a **block scope**. A scope is the range within a program for which an item (like a variable) is valid. In Rust, a variable becomes valid when it is declared, and remains valid until the scope ends at `}`. This is important because Rust does not have a **garbage collector**, nor do you manually call `free()` or `delete`.

When a variable owning heap data goes out of scope at `}`, Rust automatically calls a special function named `drop()`. Memory, file handles, network sockets, and database connections are freed instantly at the closing brace `}`.

### `println!("Hello, world!");`

* **`println!`**: This is a **macro**, not a standard function call.
* The exclamation mark (`!`) tells you that you are calling a macro.
* *Why a macro?* A standard Rust function requires a fixed number of arguments of specific types. The `println!` macro can take variable numbers of arguments and performs compile-time format-string checking (it validates that your print placeholders match your variables before your code even builds).


* **`("Hello, world!")`**: Passes a string literal argument into the macro.
* **`;`**: The semicolon marks the end of a **statement**. In Rust, statements perform an action and do not return a value.


### Compiling vs. Building with Cargo

There are two ways to run this code depending on your setup:

1. If you save the code to `main.rs`, you can compile it directly using the Rust compiler:

```bash
rustc main.rs
./main
```

`rustc` compiles the source file into a standalone machine-code executable. You can give that binary file to anyone, and they can run it without needing Rust installed.

2. In production, you create projects using Rust's package manager, **Cargo**:

```bash
cargo new hello_world
cd hello_world
cargo run
```

When you run `cargo new`, Cargo automatically generates a project folder containing a `Cargo.toml` file (manifest) and a pre-written `src/main.rs` file containing this exact "Hello, world!" snippet. `cargo run` compiles and executes it in a single step.
    
---
    
## Hello, Cargo!

In Rust, **crates**, **Cargo**, and **project structure** form the ecosystem that manages code organization, compilation, and external dependencies.

### Crates: The Unit of Compilation

A **crate** is the smallest unit of code that the Rust compiler (`rustc`) considers at one time. A crate can compile into either an executable binary or a library.

* **Binary Crate:** Compiles into an executable file that you can run. It must contain a `main()` function as its entry point.
* **Library Crate:** Compiles into code intended to be shared and reused by other programs. It does not have a `main()` function.

### Cargo: Rust’s Build System and Package Manager

`cargo` handles creating new projects, downloading external dependencies, building your code, and running tests.

#### Core Cargo Commands

```bash
cargo new my_project   # Creates a new Rust project directory
cargo build            # Compiles the project (Debug build)
cargo build --release  # Compiles with optimizations (Production build)
cargo run              # Compiles and runs binary project in one step
cargo check            # Fast check to ensure code compiles without building binaries
cargo test             # Runs unit and integration tests
```

#### Key Cargo Files

* **`Cargo.toml` (Manifest):** Defines project metadata (name, version, Rust edition) and lists third-party dependencies (from [crates.io](https://crates.io)).
```toml
[package]
name = "my_rust_app"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
description = "A brief description of what this crate does"
license = "MIT OR Apache-2.0"
repository = "https://github.com/username/my_rust_app"
readme = "README.md"
keywords = ["cli", "utility"]
categories = ["command-line-utilities"]

[dependencies]
# Standard crates.io dependency with specific features enabled
tokio = { version = "1.38", features = ["full"] }

...
```
* **`Cargo.lock`:** Auto-generated file that locks the exact versions of dependencies used to ensure reproducible builds.

### Package and Project Structure

A **package** is a collection of one or more crates defined by a single `Cargo.toml`. Standard conventions dictate where files belong within a package:

```text
my_project/
├── Cargo.toml          # Package configuration & dependencies
├── Cargo.lock          # Dependency lockfile
├── src/
│   ├── main.rs         # Root of the default Binary crate
│   ├── lib.rs          # Root of the default Library crate (optional)
│   └── models.rs       # A sub-module inside the crate
├── tests/              # Integration tests (each file here is a separate crate)
│   └── integration_test.rs
├── examples/           # Example executables showing how to use the library
│   └── basic_usage.rs
└── target/             # Compiled build outputs (ignored in git)
```