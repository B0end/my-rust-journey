# Hash Maps (`HashMap<K, V>`)

If Vectors store values by an integer index (`0`, `1`, `2`), Hash Maps store values by associating a **Key** (`K`) with a **Value** (`V`). You use the key to look up the corresponding value later.

Unlike Vectors and Strings, `HashMap` is **not included in the prelude** (the standard set of types automatically available in every file).

You must manually bring it into scope from `std::collections`:

```rust,editable
use std::collections::HashMap;

fn main() {
    // Creates a new, empty HashMap
    let mut scores = HashMap::new();

    // Insert keys (Team Name) and values (Score)
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
}
```

>[!IMPORTANT]
> Just like Vectors, all **keys must have the same type**, and all **values must have the same type**. Here, keys are `String` and values are `i32`.

## Accessing Values in a Hash Map

You look up values using the **`.get()`** method, passing a reference to the key:

```rust,editable
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");

    // `.get()` returns an `Option<&V>`
    let score: Option<&i32> = scores.get(&team_name);

    match score {
        Some(val) => println!("Blue team score: {val}"),
        None => println!("Team not found"),
    }
}
```

### Handy Shortcut: `.copied()` and `.unwrap_or()`

Instead of pattern matching every time, you can chain `.copied()` (to turn `Option<&i32>` into `Option<i32>`) and `.unwrap_or(0)` (to provide a fallback value if the key doesn't exist):

```rust
let score = scores.get(&team_name).copied().unwrap_or(0);
```

## Iterating Over a Hash Map

You can iterate over key/value pairs using a `for` loop:

```rust,editable
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{key}: {value}");
    }
}
```

## Hash Maps and Ownership

What happens to keys and values when you insert them into a Hash Map?

* Types that implement the **`Copy`** trait (like `i32`) are **copied** into the map.
* Owned types (like **`String`**) are **moved**, and the Hash Map becomes the new owner!

```rust,editable
use std::collections::HashMap;

fn main() {
    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);

    // ❌ COMPILE ERROR! `field_name` and `field_value` were MOVED into `map`!
    // println!("{field_name}"); 
}
```

If you insert references (`&field_name`) into the map instead of owned types, the values won't be moved, but **the references must remain valid for as long as the hash map lives** (Borrow Checker rules!).

## Updating a Hash Map

When inserting a key that already exists, you have three options:

### Option A: Overwrite the Existing Value

If you insert a value with an existing key, the old value is replaced:

```rust,editable
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25); // Overwrites 10 with 25!

    println!("{:?}", scores); // Prints {"Blue": 25}
}
```

### Option B: Only Insert If Key Is Absent (`entry` API)

If you only want to insert a value if the key doesn't exist yet, use the **`.entry()`** method paired with **`.or_insert()`**:

```rust,editable
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    // "Blue" exists: does nothing, returns reference to 10
    scores.entry(String::from("Blue")).or_insert(50);

    // "Yellow" does NOT exist: inserts ("Yellow", 50)
    scores.entry(String::from("Yellow")).or_insert(50);

    println!("{:?}", scores); // Prints {"Blue": 10, "Yellow": 50}
}
```

### Option C: Update a Value Based on the Old Value (Word Counter Example!)

`.or_insert()` actually returns a **mutable reference (`&mut V`)** to the value. You can dereference (`*`) it to update the value in-place!

This is famous for solving word-frequency counting:

```rust,editable
use std::collections::HashMap;

fn main() {
    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        // Find entry for `word` or insert 0 if missing. 
        // `count` holds a `&mut i32` pointing to the count inside the map!
        let count = map.entry(word).or_insert(0);

        *count += 1; // Dereference and increment in-place!
    }

    println!("{:?}", map); 
    // Prints {"world": 2, "hello": 1, "wonderful": 1}
}
```

## Summary Cheat Sheet

| Action | Syntax | Notes |
| --- | --- | --- |
| **Bring into scope** | `use std::collections::HashMap;` | Not in prelude! |
| **Create empty** | `let mut map = HashMap::new();` | Infer type on insert |
| **Insert / Overwrite** | `map.insert(key, value);` | Moves owned types |
| **Get value** | `map.get(&key)` | Returns `Option<&V>` |
| **Insert if absent** | `map.entry(key).or_insert(val);` | Idiomatic check-and-set |
| **Modify in-place** | `*map.entry(key).or_insert(0) += 1;` | Dereferences `&mut V` |
