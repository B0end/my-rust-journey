# Structs

Structs are Rust’s way of grouping related pieces of data together to build custom data types.

To define a struct, you use the `struct` keyword, name your type (using `CamelCase`), and declare its fields with their data types inside curly braces `{}`:

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

To use a struct, you create an *instance* of it by filling in every field:

```rust,editable
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        active: true,
        username: String::from("someuser123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    // Accessing fields using dot notation:
    println!("User email is: {}", user1.email);
}
```

If an instance is mutable, you can change its fields using dot notation:

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someuser123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com"); // ✅ Allowed because user1 is mut
}
```

If your function parameter names match the struct field names exactly, you don't need to type `email: email`:

```rust,editable
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username, // Equivalent to `username: username`
        email,    // Equivalent to `email: email`
        sign_in_count: 1,
    }
}
```

If you want to create a new struct instance that reuses most of another instance's values, use `..`:

```rust,editable
fn main() {
    let user1 = User {
        active: true,
        username: String::from("someuser123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    // Create user2 using user1's fields EXCEPT for email
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1 // Fills remaining fields (active, username, sign_in_count) from user1
    };
}
```

> [!WARNING]
> Look at `username` in `user2`: because `String` does **not** implement the `Copy` trait, `user1.username` was **moved** into `user2`!
> That means after this code runs, **`user1` can no longer be used as a whole variable**, because its `username` heap data was transferred! (However, `user1.email` is still valid!).

## Alternative Struct Types

### Tuple Structs (Named Tuples)

Useful when you want to give a whole tuple a distinct type name, but naming every individual field would be tedious:

```rust,editable
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);

    // Access elements using tuple index notation:
    println!("Red channel: {}", black.0);
}
```

Even though `black` and `origin` are both composed of three `i32` numbers, they are **completely distinct types**! You cannot accidentally pass a `Color` to a function expecting a `Point`.

### Unit-Like Structs (Zero Fields)

Structs that have no fields at all!

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

**Why use this?** Useful when you need to implement behavior (methods/traits) on a type, but don't need the type to store any internal data.

## Summary Table

| Struct Type | Syntax Example | When to Use? |
| --- | --- | --- |
| **Classic Struct** | `struct User { email: String }` | Named fields representing structured data |
| **Tuple Struct** | `struct Point(i32, i32, i32);` | Grouped values where field names are unnecessary |
| **Unit-Like Struct** | `struct AlwaysEqual;` | Types with behavior (traits), but zero data storage |
