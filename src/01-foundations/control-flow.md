# Control Flow

Controlling the execution flow of your code relies on two primary mechanics:
1. **Conditionals (`if` expressions)**
2. **Repetition (`loop`, `while`, and `for`)**.

## `if` Expressions

```rust,editable
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else {
        println!("number is not divisible by 4 or 3");
    }
}
```
> [!WARNING]
> Unlike C, C++, or JavaScript, Rust **will not automatically convert non-boolean types to booleans**. You must explicitly provide a `bool`.

```rust,editable
let number = 3;

// ❌ COMPILE ERROR: expected `bool`, found integer
if number { 
    println!("number was three");
}

// ✅ Correct
if number != 0 {
    println!("number was not zero");
}
```

Because `if` is an **expression** (it evaluates to a value), you can use it on the right side of a `let` statement to assign a value conditionally.

```rust,editable
fn main() {
    let condition = true;

    // Both branches MUST evaluate to the exact same type
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {number}");
}
```

## Repetition with `loop`

The `loop` keyword tells Rust to execute a block of code over and over again **forever** until you explicitly tell it to stop using `break`.

```rust,editable
fn main() {
    let mut count = 0;

    loop {
        count += 1;
        println!("count: {count}");

        if count == 3 {
            break; // Exits the loop
        }
    }
}
```

You can pass a value after the `break` keyword, and that value will be returned by the `loop` expression!

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2; // Returns 20 to `result`
        }
    };

    println!("The result is {result}"); // Prints 20
}
```

### Disambiguating Nested Loops with Loop Labels

If you have loops within loops, `break` and `continue` apply to the innermost loop by default. You can specify a **loop label** (prefixed with an apostrophe `'`) to target a specific parent loop.

```rust,editable
fn main() {
    let mut count = 0;

    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break; // Exits inner loop
            }
            if count == 2 {
                break 'counting_up; // Exits outer loop labeled 'counting_up
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```

## Conditional Loops with `while`

A `while` loop evaluates a condition before running the block. While the condition remains `true`, the loop runs; when it evaluates to `false`, the loop exits.

```rust,editable
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");
        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

## Looping Through Collections with `for`

While you can use a `while` loop to iterate through an array index by index, it is slow (due to runtime index bounds checking) and error-prone.

The `for` loop is the **most common and safest** loop construct in Rust.

```rust,editable
fn main() {
    let a = [10, 20, 30, 40, 50];

    // Safely iterate over each item in a collection
    for element in a {
        println!("the value is: {element}");
    }
}
```

To run code a specific number of times, use a `Range` provided by the standard library (`1..4` produces 1, 2, 3). You can reverse it with `.rev()`.

```rust,editable
fn main() {
    // 3, 2, 1 Countdown
    for number in (1..4).rev() {
        println!("{number}!");
    }
    println!("LIFTOFF!!!");
}
```