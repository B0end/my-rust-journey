# Functions

- You declare a function using the `fn` keyword, followed by a name, parameters inside parentheses, an optional return type, and a code block `{}`.
- Rust uses **snake_case** as the conventional style for function and variable names (e.g., `another_function`).

Parameters are special variables that are part of a function's signature. In Rust signatures, **you MUST declare the type of each parameter**.

```rust,editable
fn main() {
    print_labeled_measurement(5, 'h');
}

// Each parameter MUST have a type annotation
fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {value}{unit_label}");
}
```
When a function does not explicitly specify a return type using `->`, its return type is the **unit type**, written as **`()`**.

So, both `print_labeled_measurement` and `main` have a return type of **`()`**:

```rust,editable
fn main() -> (){
    print_labeled_measurement(5, 'h');
}

// Each parameter MUST have a type annotation
fn print_labeled_measurement(value: i32, unit_label: char) -> () {
    println!("The measurement is: {value}{unit_label}");
}
```

## Statements vs. Expressions

| Concept | Definition | Has a return value? | Ends with a semicolon `;`? |
| --- | --- | --- | --- |
| **Statement** | Instructions that **perform an action** and do not return a value. | ❌ No (returns `()`) | Yes `;` |
| **Expression** | Evaluates to a **resulting value**. | ✅ Yes | No `;` |

### Statements

```rust,editable
let y = 6; // Statement
```

Because statements do not return values, **you cannot assign a `let` statement to another variable** in Rust (unlike C or Python):

```rust,editable
let x = (let y = 6); // ❌ COMPILE ERROR: `let` is a statement, not an expression
```

### Expressions

- Expressions evaluate to a value:
    - Calling a function.
    - Calling a macro (`println!`).
    - Performing math (`5 + 6`).
    - Creating a new scope block `{}` with `{ let x = 3; x + 1 }`.

Notice the lack of a semicolon on the last line of an expression block:

```rust,editable
fn main() {
    let y = {
        let x = 3;
        x + 1 // 👈 NO SEMICOLON! This evaluates to 4 and is returned to `y`
    };

    println!("The value of y is: {y}"); // Prints 4
}
```

## Return Values

- Functions can return values to the code that calls them.
    - Declare return types using an arrow `->`.
    - The return value of a function is synonymous with the value of the **final expression** in the block.
    - You can return early from a function using the `return` keyword and a semicolon (useful for early exits/guards).

```rust,editable
fn five() -> i32 {
    5 // Final expression: no `return` keyword, no semicolon!
}

fn plus_one(x: i32) -> i32 {
    x + 1 // Evaluates to x + 1 and returns it
}

fn main() {
    let x = five();
    let y = plus_one(x);
    println!("y is: {y}"); // 6
}
```