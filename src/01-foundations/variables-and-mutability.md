# Variables and Mutability

| Feature | `let x = ...` | `let mut x = ...` | `const X: T = ...` |
| --- | --- | --- | --- |
| **Reassignable?** | No | Yes | No |
| **Type change?** | Via shadowing | No | No |
| **Evaluation** | Runtime | Runtime | Compile-time |
| **Scope** | Block scope | Block scope | Global / any scope |
| **Type Annotation** | Optional (inferred) | Optional (inferred) | **Required** |

## Immutable vs. Mutable Variables

By default, variable bindings in Rust are **immutable**. Once bound to a name, a value cannot be changed.

Compile Error:

```rust,editable
fn main() {
    let x = 5;
    x = 6;
}
```

So to allow reassignment, explicitly mark the binding with `mut`.

```rust,editable
fn main() {
    let mut x = 5;
    println!("x is: {x}"); // 5
    x = 6;                 // ✅ Allowed
    println!("x is: {x}"); // 6
}
```

## Variable Shadowing

Shadowing occurs when you declare a new variable using the `let` keyword with the **same name** as a previous variable in scope.

```rust,editable
fn main() {
    let x = 5;
    let x = x + 1; // Shadows the previous `x` (x is now 6)

    {
        let x = x * 2; // Shadows `x` only inside this inner scope (x is 12)
        println!("Inner x: {x}"); // 12
    }

    println!("Outer x: {x}"); // 6 (inner shadowing ended)
}
```

- You can apply transformations to a variable without leaving it permanently mutable. Once the `let` chain ends, the variable remains immutable.
- Unlike `mut`, shadowing creates a brand-new variable, allowing you to change the underlying data type.

```rust
// ✅ Valid with Shadowing
let spaces = "   ";          // Type: &str
let spaces = spaces.len();   // Type: usize

// ❌ Fails with `mut`
let mut spaces = "   ";
spaces = spaces.len();       // Compile Error: mismatched types (&str vs usize)
```

* Use **`mut`** when you want to modify a value in-place over time (e.g., an accumulator or loop counter).
* Use **Shadowing** when you want to transform a value (or change its type) while keeping the resulting variable immutable afterward.
* Use **`const`** for hardcoded, global, or compile-time invariant values (see next).


## Constants (`const`)

- Constants can only use operations that Rust can evaluate at compile-time (like basic arithmetic or const fn), not runtime function calls or heap allocations.

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```


## Compile-Time vs. Runtime 

Think of building a Rust application like baking a cake:

**Compile-Time**: The phase when you read the recipe, gather ingredients, and prep everything before putting it in the oven. If the recipe calls for salt and you accidently grab soap, you catch the mistake before anyone eats it.

**Runtime**: The moment the user actually eats the cake. If there's a problem that only reveals itself when eaten (e.g., the user is allergic to peanuts), that happens live at runtime.

### Compile-Time (`cargo check` or `cargo build`)

This is when the Rust compiler (`rustc`) reads your source code and translates it into machine code (an executable `.exe` or binary file).

* **What happens:** Type checking, syntax checking, constant math evaluation, memory safety checks.
* **Who is involved:** You (the developer) and the Rust compiler on your computer.
* **The User experience:** The end-user **never sees this phase**.

In Rust, `const` values **MUST** be fully calculated at compile-time.

```rust,editable
// ✅ COMPILE-TIME: The compiler does the math (60 * 60 * 3 = 10800) 
// while turning your code into a executable.
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

The compiler calculates `10800` ahead of time and hardcodes that single number directly into the final program binary. When the program runs, zero CPU math is needed.

### Runtime (`cargo run` or running the `.exe`)

This is when your compiled program is actually running on a computer or server, responding to real-world input.

* **What happens:** User input, reading files, networking, dynamic calculations, user interaction.
* **Who is involved:** The user interacting with your software.
* **The User experience:** The software is actively running.

Regular variables (`let` / `let mut`) store values that are calculated or loaded while the program is actively running.

```rust,editable
use std::io;

fn main() {
    println!("Please enter your name:");

    // ⚡ RUNTIME: The computer has to stop and wait for a human user 
    // to type something into the terminal. 
    // The compiler CANNOT know what the user will type beforehand!
    let mut user_input = String::new();
    io::stdin().read_line(&mut user_input).unwrap();
}
```

> [!NOTE]
> - If Rust can figure it out **before the program starts running**, it's **Compile-Time**.
> - If Rust has to wait for **the program to actually run** to get the value, it's **Runtime**.
