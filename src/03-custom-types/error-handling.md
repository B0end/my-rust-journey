# Error Handling

Rust separates errors into two distinct categories so you always know how to handle them cleanly.

1. **Unrecoverable Errors (`panic!`)**: Something went terribly wrong, and the program must stop immediately (e.g., accessing an index beyond the end of an array).
2. **Recoverable Errors (`Result<T, E>`)**: Something went wrong, but it's expected and your program can handle it gracefully (e.g., file not found, network request timed out).

## Unrecoverable Errors with `panic!`

When the `panic!` macro executes, your program prints a failure message, unwinds (cleans up) the stack, and exits immediately.

You can trigger a panic manually:

```rust,editable
fn main() {
    panic!("crash and burn"); // Program aborts here immediately!
}
```

Panics often happen due to bugs in your code, such as attempting to access an invalid array index:

```rust,editable
fn main() {
    let v = vec![1, 2, 3];

    v[99]; // 💥 PANIC! `index out of bounds: the len is 3 but the index is 99`
}
```

Unlike C or C++, where reading out-of-bounds causes dangerous **buffer overreads** (reading random RAM memory), Rust stops execution immediately to protect your memory safety.

To find out exactly what line of code caused a panic, set the environment variable **`RUST_BACKTRACE=1`** when running your program in the terminal:

```bash
$ RUST_BACKTRACE=1 cargo run
```

This prints a full **backtrace** (the list of all functions called up until the crash) so you can pinpoint the exact source file and line number.

## Recoverable Errors with `Result<T, E>`

Most errors aren't serious enough to warrant shutting down the entire program. For example, if opening a file fails, you might want to create the file or prompt the user for a different path.

Rust handles recoverable errors using the **`Result<T, E>`** enum:

```rust
enum Result<T, E> {
    Ok(T),  // The operation succeeded; holds return value `T`
    Err(E), // The operation failed; holds error details `E`
}
```

### Handling `Result` with `match`

Let's attempt to open a file using `std::fs::File::open`:

```rust,editable
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {error:?}"),
    };
}
```

### Handling Different Types of Errors

What if we want to create the file if it doesn't exist, but panic if any other error occurs? We can nest `match` expressions or check the error's kind:

```rust,editable
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {e:?}"),
            },
            other_error => {
                panic!("Problem opening the file: {other_error:?}");
            }
        },
    };
}
```

## Shortcuts for Panic on Error: `unwrap` and `expect`

Matching every `Result` with a `match` statement can get verbose. Rust provides convenience methods:

- `.unwrap()`
    - If `Result` is `Ok`, it returns the value inside `Ok`.
    - If `Result` is `Err`, it calls `panic!` for you automatically.

```rust
let greeting_file = File::open("hello.txt").unwrap();
```

- `.expect("custom message")`
    - Works like `.unwrap()`, but allows you to pass a **custom panic message**, making debugging significantly easier.

```rust
let greeting_file = File::open("hello.txt")
    .expect("hello.txt should be included in this project directory!");
```

> [!TIP]
> **Best Practice:** Prefer `.expect()` over `.unwrap()` in production code so your log outputs contain helpful error context!

## Propagating Errors with the `?` Operator

When writing a function that might fail, instead of handling the error inside the function, you often want to **pass (propagate) the error back to the caller**.

Verbose Way (using `match`):

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e), // Propagate error early!
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e), // Propagate error early!
    }
}
```

The **`?` operator** eliminates all that boilerplate!

```rust,editable
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?; // 👈 If Err, returns early automatically!
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;    // 👈 If Err, returns early automatically!
    Ok(username)
}
```

Even shorter (Chaining with `?`):

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();
    File::open("hello.txt")?.read_to_string(&mut username)?;
    Ok(username)
}
```

> [!IMPORTANT]
> You can only use the `?` operator inside functions whose return type is **compatible** with the value you are applying `?` to (e.g., functions returning `Result` or `Option`).

## To `panic!` or Not to `panic!`

How do you decide when to return `Result` vs when to call `panic!`?

| Situation | Choice | Reason |
| --- | --- | --- |
| **Examples, Prototypes, Tests** | `panic!` / `unwrap()` | Keeps code short and focused on demonstrating concepts. |
| **Parsing user input / Network calls** | `Result` | Failure is expected; caller should handle it gracefully. |
| **Invalid internal state (Bugs)** | `panic!` | Program assumptions are violated (e.g., negative numbers passed where prohibited). |
| **Contract violations** | `panic!` | Caller passed invalid parameters violating function contract. |

## Summary Cheat Sheet

| Syntax | What it does |
| --- | --- |
| **`panic!("msg")`** | Aborts program immediately with message |
| **`Result<T, E>`** | Enum representing success (`Ok(T)`) or failure (`Err(E)`) |
| **`res.unwrap()`** | Returns `T` on `Ok`, panics on `Err` |
| **`res.expect("msg")`** | Returns `T` on `Ok`, panics with `"msg"` on `Err` |
| **`expr?`** | Returns `T` on `Ok`, returns `Err` from enclosing function on `Err` |
