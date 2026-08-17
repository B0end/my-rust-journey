# References and Borrowing

Now that you have a rock-solid grasp on **Ownership**, you might be thinking:

> *"Do I really have to keep giving away ownership or cloning my data every single time I pass a variable into a function?"*

If you had to pass ownership into every function and return it back every single time, writing Rust code would be an exhausting game of hot potato:

```rust
// ❌ ANNOYING WAY: Passing ownership back and forth manually
fn main() {
    let s1 = String::from("hello");
    let (s2, len) = calculate_length(s1); // We had to return s1 back as s2!
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len();
    (s, length) // Returning ownership back
}
```

Instead of giving someone your book permanently (a **Move**), or buying them a whole new copy of the book (a **Clone**), you simply **let them borrow it**.

* **References (`&`)** are like giving someone permission to **read** your library book.
* **Borrowing** is the action of creating a reference.
* When they are done reading it, they hand it back. **You never lose ownership!**

```rust,editable
// ✅ ELEGANT WAY: Borrowing with a Reference
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1); // We pass a reference (&s1)

    println!("The length of '{s1}' is {len}."); // ✅ s1 is still alive and valid!
}

fn calculate_length(s: &String) -> usize {
    s.len()
} // Here, `s` goes out of scope. But because it doesn't OWN the String, nothing is dropped!
```

1. During `calculate_length(&s1)` Execution:

```text
               STACK MEMORY                                          HEAP MEMORY
+------------------------------------------+          +---------------------------------------+
|  [main Frame]                            |          | Address | Value                       |
|    Address 1000: s1                      |          | 3000    | 'h'                         |
|      ptr -----------------------------------------> | 3001    | 'e'                         |
|      len: 5                              |          | 3002    | 'l'                         |
|      cap: 5                              |          | 3003    | 'l'                         |
+------------------------------------------+          | 3004    | 'o'                         |
|  [calculate_length Frame]                |          +---------------------------------------+
|    Address 1024: s                       |
|      ptr -----------------------------> Address 1000 (s1 on Stack)
+------------------------------------------+
  ^
  STACK POINTER (SP) = 1032
```

> [!NOTE]
> Notice the two-step jump (Indirection):
> * `s` (Address `1024`) points to `s1` (Address `1000`) on the Stack.
> * `s1` (Address `1000`) points to `"hello"` (Address `3000`) on the Heap.

![Memory reference diagram](../trpl04-06.svg)

2. `calculate_length` Finishes (Stack Frame Popped). The CPU moves the Stack Pointer (`SP`) back up to `1024`.

```text
               STACK MEMORY                                          HEAP MEMORY
+------------------------------------------+          +---------------------------------------+
|  [main Frame]                            |          | Address | Value                       |
|    Address 1000: s1                      |          | 3000    | 'h'                         |
|      ptr -----------------------------------------> | 3001    | 'e'                         |
|      len: 5                              |          | 3002    | 'l'                         |
|      cap: 5                              |          | 3003    | 'l'                         |
+------------------------------------------+          | 3004    | 'o'                         |
|  [POPPED FRAME / UNUSED SPACE]           |          +---------------------------------------+
|    Address 1024: s (Ghost Data)          |
+------------------------------------------+
  ^
  STACK POINTER (SP) = 1024
```

---

## Immutable References (`&T`) — "Read-Only"

You can look at the data, but you **cannot modify it**.

```rust,editable
fn main() {
    let s = String::from("hello");
    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world"); // ❌ COMPILE ERROR! Cannot mutate immutable reference
}
```

## Mutable References (`&mut T`) — "Read & Write"

If you want to modify the borrowed data, you must declare the variable as `mut` AND create a mutable reference (`&mut`).

```rust,editable
fn main() {
    let mut s = String::from("hello"); // Variable MUST be mutable

    change(&mut s); // We pass a MUTABLE reference

    println!("{s}");
}

fn change(some_string: &mut String) { // Accepts a mutable reference
    some_string.push_str(", world");
}
```

> [!CAUTION]
> - Cannot borrow `s` as mutable more than once at a time:

```rust,editable
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s; // ❌ COMPILE ERROR

println!("{r1}, {r2}");
```

> [!CAUTION]
> - Cannot borrow `s` as mutable because it is also borrowed as immutable. If you have readers (`&s`), the data is **frozen**. No one can mutate it until all readers are done using it!

```rust,editable
let mut s = String::from("hello");

let r1 = &s; // 🤓 Reader 1: "I'm looking at 'hello'"
let r2 = &s; // 🤓 Reader 2: "I'm looking at 'hello'"
let r3 = &mut s; // ❌ COMPILE ERROR

println!("{r1}, {r2}, {r3}");
```

A reference's scope doesn't always last until the end of the curly brace `}`. It lasts from where it is created **up to the point it is LAST USED**.

```rust,editable
let mut s = String::from("hello");

let r1 = &s; // 🤓 Read-only borrow begins
let r2 = &s; 
println!("{r1} and {r2}"); // 🛑 r1 and r2 are LAST USED here!

// Since r1 and r2 are never used again below this line, their borrow ENDS HERE!

let r3 = &mut s; // ✅ WORKS! No active readers exist anymore.
println!("{r3}");
```

## Preventing Dangling References

In C/C++, it's easy to return a pointer to memory that has already been deleted (a **Dangling Pointer**). Rust makes this impossible at compile time!

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // We promise to return a reference to a String
    let s = String::from("hello"); // s is created inside dangle()

    &s // ❌ We return a reference to `s`
} // `s` goes out of scope and is DROPPED! Memory is freed!
  // Returning `&s` would point to DEAD memory!
```

**The Fix:** Return the `String` directly to **move ownership** out of the function:

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    s // ✅ Move ownership out cleanly!
}
```

## Summary Cheat Sheet

| Feature | Syntax | Can Read? | Can Modify? | How Many Allowed Simultaneously? |
| --- | --- | --- | --- | --- |
| **Immutable Reference** | `&s` | ✅ Yes | ❌ No | Infinite (as long as 0 writers exist) |
| **Mutable Reference** | `&mut s` | ✅ Yes | ✅ Yes | Exactly ONE (and 0 readers) |
