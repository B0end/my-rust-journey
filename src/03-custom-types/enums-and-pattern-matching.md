# Enums and Pattern Matching

If Structs allow you to group multiple values together (**"AND"** types—a user has an email *AND* a username), Enums allow you to express a value that is **one of several possible variants** (**"OR"** types—an IP address is IPv4 *OR* IPv6).

An **Enum** (short for *enumeration*) lets you define a type by listing its possible **variants**.

```rust,editable
enum Direction {
    North,
    South,
    East,
    West,
}

fn main() {
    let four = Direction::North; // Access variants using `::`
}
```

A variable of type `Direction` can only ever hold **one** of these four values at a time.

## Attaching Data directly to Variants

```rust,editable
enum Message {
    Quit,                       // Holds no data
    Move { x: i32, y: i32 },    // Holds an anonymous struct
    Write(String),              // Holds a single String
    ChangeColor(i32, i32, i32), // Holds three i32 tuple values
}
```

## Implementing Methods on Enums (`impl`)

Just like Structs, you can define methods on Enums using an `impl` block:

```rust,editable
impl Message {
    fn call(&self) {
        // Method body to execute on a Message variant
    }
}

fn main() {
    let m = Message::Write(String::from("hello"));
    m.call();
}
```

## The `Option` Enum (Rust's Null Replacement)

In many programming languages, variables can have a special value called `null` or `nil`, representing "no value". **Rust does NOT have `null`**. Instead, Rust encodes the concept of a value being present or absent using a standard library enum called **`Option<T>`**:

```rust
enum Option<T> {
    None,    // Represents "no value"
    Some(T), // Represents a value of type T exists
}
```

```rust,editable
fn main() {
    let some_number: Option<i32> = Some(5);
    let absent_number: Option<i32> = None;

    let x: i32 = 5;

    // ❌ COMPILE ERROR! Cannot add `i32` and `Option<i32>` directly!
    // let sum = x + some_number; 
}
```

Because `Option<i32>` and `i32` are completely **different types**, the compiler forces you to handle the `None` case **before** you can access the value inside `Some(T)`.

You can *never* accidentally use a missing value as if it exists!

## The `match` Control Flow Construct

How do we actually get the value out of an Enum or an `Option`?

We use **Pattern Matching** with `match`.

Think of `match` as an extremely powerful, compiler-checked `switch` statement.

```rust,editable
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

When enum variants hold data, `match` allows you to **bind variables to the inner data**:

```rust,editable
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(String), // Quarter holds the state name (e.g. "Alaska")
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => { // Binds `state` to the string inside Quarter
            println!("State quarter from {state}!");
            25
        }
    }
}
```

## Matching with `Option<T>`

`match` is the primary way to safely unpack an `Option`:

```rust,editable
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1), // Extract `i`, add 1, wrap back in Some
    }
}

fn main() {
    let five = Some(5);
    let six = plus_one(five); // Returns Some(6)
    let none = plus_one(None); // Returns None
}
```

## Matches Are Exhaustive!

This is a key rule in Rust: **`match` expressions must cover every single possible case!**

If you leave out a variant, your code **will not compile**:

```rust,editable
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1), // ❌ COMPILE ERROR: pattern `None` not covered!
    }
}
```

### The Catch-All Pattern (`_`)

If you don't want to list every individual variant, use `other` to capture the remaining value or `_` to ignore it:

```rust,editable
let dice_roll = 9;

match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    _ => rerun_turn(), // Matches ANYTHING else!
}
```

## Concise Control Flow with `if let`

Verbose Way (`match`):

```rust
let config_max = Some(8u8);

match config_max {
    Some(max) => println!("The maximum is configured to be {max}"),
    _ => (),
}
```
Concise Way (`if let`):

```rust,editable
let config_max = Some(8u8);

// "If config_max matches the pattern Some(max), run this block:"
if let Some(max) = config_max {
    println!("The maximum is configured to be {max}");
}
```

`if let` acts like syntax sugar for a `match` that runs code for one specific pattern and ignores everything else. You can also append an `else` block if needed!

## Summary Cheat Sheet

| Concept | Syntax | What it does |
| --- | --- | --- |
| **Enum** | `enum Status { Active, Idle }` | Type with fixed set of possibilities |
| **Enum with Data** | `enum Value { Number(i32) }` | Variants holding embedded data |
| **Option** | `Option<T> { Some(T), None }` | Safe alternative to `null` |
| **`match`** | `match x { Pattern => code }` | Pattern matching for every variant (Exhaustive!) |
| **`if let`** | `if let Pattern = value { ... }` | Short-hand matching for a single variant |
