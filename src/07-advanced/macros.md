# Macros

* Frequently Used Built-in Macros
    * **`println!` / `format!`:** String formatting and output.
    * **`vec!`:** Concise creation of a `Vec<T>`.
    * **`panic!` / `assert!`:** Error handling and testing assertions.
    * **`todo!` / `unimplemented!`:** Placeholders for unfinished code paths that still pass type checking.

## Key Advantages over Functions

* **Variadic Arguments:** Functions in Rust require a fixed number of parameters. Macros like `println!` or `vec!` can take any number of arguments.
* **Compile-Time Execution:** Code generation happens during compilation, adding zero runtime overhead.
* **Domain-Specific Languages (DSLs):** Macros let you define custom syntax within Rust (e.g., embedding SQL queries or JSON structures).

## The Two Types of Macros

Rust provides two distinct macro systems:

### 1. Declarative Macros (`macro_rules!`)

Declarative macros match against code patterns and replace them with new code. They function similarly to a `match` expression operating on Rust source code.

```rust
// A simplified custom implementation of the vec! macro
macro_rules! my_vec {
    // LEFT SIDE: Matcher Pattern
    // Matches expressions ($x:expr) separated by commas (,*)
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            
            // RIGHT SIDE: Repetition Expansion
            // Repeats `temp_vec.push($x);` for every $x matched
            $(
                temp_vec.push($x);
            )*
            
            temp_vec
        }
    };
}

fn main() {
    // Expands at compile time into:
    // let mut temp_vec = Vec::new(); temp_vec.push(1); temp_vec.push(2); temp_vec
    let v = my_vec![1, 2, 3];
}
```

* **Syntax Pattern:** `$( $x:expr ),*` captures zero or more comma-separated expressions.
* **Repetition:** The `$( ... )*` block repeats the code for each matched expression.

### 2. Procedural Macros

Procedural macros accept a stream of Rust tokens as input, run arbitrary Rust code to transform those tokens, and output a new token stream. They act like functions that run during compilation.

Procedural macros must be defined in their own crate type (`proc-macro`) and fall into three categories:

| Category | Typical Usage | Example |
| --- | --- | --- |
| **Custom Derive** | Automatically implement traits for structs/enums | `#[derive(Serialize, Deserialize)]` |
| **Attribute-like** | Define custom attributes attached to items | `#[route(GET, "/")]` |
| **Function-like** | Custom syntax called via exclamation mark | `sql!("SELECT * FROM users")` |

### Example: A Custom `Hello` Derive Macro

The most common and illustrative type is a **Custom Derive Macro**, which automatically implements a trait for a struct or enum.

To build a procedural macro, you must define it in a separate crate with the `proc-macro = true` flag set in its `Cargo.toml`.

#### 1. Project Setup

Create two crates in a workspace:

1. `hello_macro` (a standard library crate that defines the trait)
2. `hello_macro_derive` (the procedural macro crate)

#### 2. Declare the Trait (`hello_macro/src/lib.rs`)

```rust
pub trait HelloMacro {
    fn hello_macro();
}
```

#### 3. Define the Derive Macro (`hello_macro_derive/Cargo.toml`)

```toml
[package]
name = "hello_macro_derive"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true

[dependencies]
syn = "2.0"    # Parses Rust code into a syntax tree (AST)
quote = "1.0"  # Converts syntax trees back into Rust code tokens
```

#### 4. Implement the Macro (`hello_macro_derive/src/lib.rs`)

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // 1. Parse the input Rust code into an AST
    let ast = parse_macro_input!(input as DeriveInput);

    // 2. Extract the name of the struct or enum
    let name = &ast.ident;

    // 3. Generate the implementation using the quote! macro
    let expanded = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };

    // 4. Convert the generated code back into a TokenStream
    TokenStream::from(expanded)
}
```

#### 5. Use the Macro in Your Application

Add `hello_macro` and `hello_macro_derive` to your `Cargo.toml` dependencies, then annotate your types:

```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
    // Output: Hello, Macro! My name is Pancakes!
}
```

### Key Takeaways

* **TokenStream:** The raw input and output of procedural macros.
* **`syn` crate:** Parses `TokenStream` into an Abstract Syntax Tree (AST) so you can inspect fields, types, and identifiers.
* **`quote` crate:** Turns macro-generated template code back into a `TokenStream` using variable interpolation (like `#name`).
* **Isolation requirement:** Procedural macros must reside in their own dedicated crate with `proc-macro = true`.