# UTF-8 Encoded Strings

Strings in Rust can feel surprisingly complex because Rust forces you to handle the raw reality of **UTF-8 encoding** upfront so your code never crashes on international text, emojis, or special characters.

Rust actually has two main string types:

* **`&str` (String Slice):** A borrowed view into a sequence of UTF-8 encoded bytes stored somewhere (in binary memory, on the stack, or on the heap).
* **`String`:** A growable, mutable, heap-allocated, owned UTF-8 byte buffer.

> [!NOTE]
> Under the hood, a `String` is actually just a **wrapper around a vector of bytes (`Vec<u8>`)** with a guarantee that the bytes are always valid UTF-8!

## Creating a String

Many of the same operations available with `Vec<T>` work with `String` because `String` is built on top of a vector.

```rust,editable
fn main() {
    // 1. Creating an empty String
    let mut s1 = String::new();

    // 2. Converting a string literal (&str) into a String using .to_string()
    let data = "initial contents";
    let s2 = data.to_string();

    // 3. Using String::from() (Equivalent to .to_string())
    let s3 = String::from("initial contents");
}
```

## Updating a String

A `String` can grow in size and its contents can change, just like a `Vec<i32>`.

* `push_str(&str)` appends a string slice (does **not** take ownership!).
* `push(char)` appends a single character.

```rust,editable
fn main() {
    let mut s = String::from("foo");
    s.push_str("bar"); // `s` is now "foobar"

    let mut s2 = String::from("lo");
    s2.push('l');      // `s2` is now "lol"
}
```

```rust,editable
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");

    // Using `+` operator
    let s3 = s1 + &s2; // ⚠️ `s1` has been MOVED and can no longer be used!
                       // `&s2` is borrowed.
}

```

> [!WARNING]
> **Why did `s1` get moved?**
> The `+` operator uses a function that looks like `fn add(self, s: &str) -> String`. It takes **ownership** of `s1`, appends a copy of `s2`'s contents to it, and returns `s1`. This avoids making two full heap allocations!

If you need to concatenate multiple strings without losing ownership, use the **`format!` macro**:

```rust,editable
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    // `format!` works like `println!`, but returns a new String!
    // It does NOT take ownership of any variables.
    let s = format!("{s1}-{s2}-{s3}"); // "tic-tac-toe"
}
```

## Indexing into Strings: Why Rust Prohibits It!

In Python or C++, you can access characters with `s[0]`. If you try that in Rust, **it will not compile!**

```rust,editable
let s = String::from("hello");
// let h = s[0]; // ❌ COMPILE ERROR! `String` cannot be indexed by integer!
```

Because **UTF-8 characters vary in byte length** (1 to 4 bytes per character)!

Look at these two words:

1. `"hello"` ➡️ 5 characters, **5 bytes** long (each letter is 1 byte).
2. `"Здравствуйте"` (Russian) ➡️ 12 characters, **24 bytes** long (each letter is 2 bytes!).
3. `"🦀"` (Emoji) ➡️ 1 character, **4 bytes** long!

If Rust allowed `s[0]` on `"Здравствуйте"`, you wouldn't get `'З'`. You would get `208` (the first raw byte of a 2-byte character), which is invalid Unicode on its own!

### Three Ways Rust Sees Strings

To Rust, a string like `"Зд"` can be viewed in three distinct ways:

1. **Bytes:** `[208, 151, 208, 180]` (Raw 8-bit integers in memory).
2. **Scalar Values (`char`):** `['З', 'д']` (Unicode code points).
3. **Grapheme Clusters:** `"З"`, `"д"` (What humans call "letters").

## How to Safely Walk Through Strings

If you need to iterate over string content, be explicit about whether you want **characters** or **raw bytes**:

- **Option A**: Iterate over `char` (Unicode Scalar Values)

```rust,editable
fn main() {
    for c in "Зд".chars() {
        println!("{c}");
    }
}
// Output:
// З
// д
```

- **Option B**: Iterate over raw `u8` Bytes

```rust,editable
fn main() {
    for b in "Зд".bytes() {
        println!("{b}");
    }
}
// Output:
// 208
// 151
// 208
// 180
```

## String Slicing (`&s[start..end]`)

You *can* slice a string using ranges, but **the range indices MUST align with UTF-8 character boundaries**:

```rust,editable
fn main() {
    let hello = "Здравствуйте";

    // Grab first 4 bytes (since each character is 2 bytes, this gets "Зд")
    let s = &hello[0..4]; 
    println!("{s}"); // Prints "Зд"

    // ⚠️ DANGER: What if you slice at byte index 1?
    // let bad_slice = &hello[0..1]; // 💥 PANIC AT RUNTIME! Crash!
}
```

> [!CAUTION]
> Slicing in the middle of a multi-byte character will cause your program to **crash at runtime**. Use string slicing with extreme caution!

## Summary Cheat Sheet

| Task | Code Example | Notes |
| --- | --- | --- |
| **New Empty String** | `String::new()` | Heap-allocated buffer |
| **Convert `&str` to `String`** | `String::from("hi")` | Allocates on heap |
| **Append String Slice** | `s.push_str("bar")` | Does not take ownership |
| **Append Single Char** | `s.push('!')` | Appends 1 `char` |
| **Format Strings** | `format!("{s1}-{s2}")` | Doesn't consume inputs |
| **Iterate Characters** | `for c in s.chars()` | Safe Unicode iteration |
| **Iterate Bytes** | `for b in s.bytes()` | Raw `u8` byte iteration |
