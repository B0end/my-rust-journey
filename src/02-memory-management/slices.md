# The Slice Type

A **slice** is a **view into a portion of a data structure**. It's like taking a pair of scissors and cutting out a reference to a window of data.

Slices let you reference a contiguous sequence of elements in a collection. A slice is a kind of reference, so it does not have ownership.

```rust,editable
fn main() {
    let s = String::from("hello world");

    // These syntax ranges are all a slice!
    let hello = &s[0..5];  // Grab bytes 0 through 4 ("hello")
    let world = &s[6..11]; // Grab bytes 6 through 10 ("world")

    let hello_world = format!("{}, {}", hello, world);
    println!("{hello_world}");
}
```

`&s[0..5]` means *"give me a reference to characters from index 0 up to (but not including) 5"*. `[start..end]`

Why do we need `&str` instead of `&String`?

* `String` is the **owner** sitting on the Stack, holding a ticket to heap memory.
* `&String` is a reference to the **entire** string.
* `&str` (String Slice) is a reference to **any piece** of string memory (a start pointer + length).

```rust,editable
fn main(){
    let s = String::from("hello");
    
    let full_ref: &String = &s;      // Points to the whole String struct
    let slice_ref: &str   = &s[0..2]; // Points directly to "he" (starts at 'h', length 2)
    
    println!("{full_ref}");
    println!("{slice_ref}");
}
```

| Syntax | Meaning | Example |
| --- | --- | --- |
| `&s[0..2]` | Start at 0, go up to 2 | `"he"` |
| `&s[..2]` | **Shortcut:** From beginning up to 2 | `"he"` |
| `&s[3..5]` | Start at 3 up to 5 | `"lo"` |
| `&s[3..]` | **Shortcut:** From 3 to the very end | `"lo"` |
| `&s[..]` | **Shortcut:** Slice the whole string | `"hello"` |

## What is the difference between `&String` and `&str`?

Think back to your restaurant kitchen analogy:

* **`String`**: You own the heap storage space. You hold the full ticket (`ptr`, `len`, `capacity`).
* **`&String`**: You borrow the **entire ticket** sitting on someone else's desk. You are pointing to the `String` owner variable itself.
* **`&str` (String Slice)**: You don't care about the owner's ticket. You get a **direct view window** into a specific section of raw characters in the heap warehouse (`ptr`, `len`).

| Type | What is it? | What information does it hold? | How many bytes on the Stack? |
| --- | --- | --- | --- |
| **`&String`** | Reference to the entire `String` owner | **1 Pointer** (points to `s1` on the Stack) | **8 bytes** (1 pointer) |
| **`&str`** | Slice / Window into string data | **1 Pointer** (points to RAM) + **1 Length** | **16 bytes** ("Fat Pointer") |

> [!NOTE]
> - Standard pointers (`&String`) only store **1 item**: an address (8 bytes).
> - A slice (`&str`) stores **2 items**: a starting address + length (16 bytes). Because it holds extra information, Rust developers call it a **"Fat Pointer"**!


## Graphical Memory Layout

```rust,editable
fn main() {
    let s1 = String::from("hello world");

    let full_ref: &String = &s1;        // Reference to the whole String
    let slice_ref: &str   = &s1[0..5];  // Slice containing "hello"
}
```

```text
               STACK MEMORY                                          HEAP MEMORY
+------------------------------------------+          +---------------------------------------+
|  Address 1000: s1 (String owner)         |          | Address | Value                       |
|    ptr -------------------------------------------> | 3000    | 'h'                         |
|    len: 11                               |          | 3001    | 'e'                         |
|    cap: 11                               |          | 3002    | 'l'                         |
+------------------------------------------+          | 3003    | 'l'                         |
|  Address 1024: full_ref (&String)        |          | 3004    | 'o'                         |
|    ptr -----------------------------> Address 1000  | 3005    | ' '                         |
+------------------------------------------+          | 3006    | 'w'                         |
|  Address 1032: slice_ref (&str)          |          | ...     | ...                         |
|    ptr -------------------------------------------> | 3000 ('h')                             |
|    len: 5                                |          +---------------------------------------+
+------------------------------------------+
```

1. **`full_ref` (`&String` at Address `1024`):**
    * Points to **Address `1000` on the Stack** (where `s1` lives).
    * It takes a two-step jump (indirection) to reach the heap: `full_ref` ➡️ `s1` ➡️ Heap Address `3000`.
2. **`slice_ref` (`&str` at Address `1032`):**
    * Points **directly to Heap Address `3000`** where `'h'` sits!
    * It stores its own length (`5`). It doesn't care that `s1` has length `11`—its window only spans bytes `3000` through `3004`.

What if we created a slice for `"world"` instead?

```rust
let world_slice: &str = &s1[6..11];
```

```text
               STACK MEMORY                                          HEAP MEMORY
+------------------------------------------+          +---------------------------------------+
|  Address 1000: s1 (String owner)         |          | Address | Value                       |
|    ptr -------------------------------------------> | 3000    | 'h'                         |
|    len: 11                               |          | ...     | ...                         |
|    cap: 11                               |          | 3005    | ' '                         |
+------------------------------------------+          | 3006    | 'w'                         |
|  Address 1040: world_slice (&str)        |          | 3007    | 'o'                         |
|    ptr -------------------------------------------> | 3006 ('w')                             |
|    len: 5                                |          | ...     | ...                         |
+------------------------------------------+          +---------------------------------------+
```

Look at that `ptr` inside `world_slice`: it simply points to **Address `3006`** (where `'w'` starts) with a length of `5`.

<img src="../trpl04-07.svg" alt="A string slice referring to part of a String" width="300" />

---

> [!NOTE]
> Instead of thinking of array indices as "boxes" holding items, think of memory like a standard ruler with numbered tick marks.

```text
Memory Addresses:   3000   3001   3002   3003   3004   3005   3006   3007   3008   3009   3010   3011
Tick Marks:           |------|------|------|------|------|------|------|------|------|------|------|
Characters:           | 'h'  | 'e'  | 'l'  | 'l'  | 'o'  | ' '  | 'w'  | 'o'  | 'r'  | 'l'  | 'd'  |
Index Offsets:        0      1      2      3      4      5      6      7      8      9      10     11
```

1. **Why Start at 0?**

Index `0` doesn't mean "Item #1." It means **"How many steps away from the start?"**
* `'h'` is at the very beginning, so it is **0 steps** from the start.
* `'w'` is **6 steps** away from the start (address `3000 + 6 = 3006`).

2. **Why End Minus Start Gives the Length (`end - start`)**

When Rust slices `[0..5]`, the numbers `0` and `5` are the **tick marks on the ruler**, not the characters themselves.
* Start tick: `0`
* End tick: `5`
* Distance between tick 0 and tick 5: `5 - 0 = 5` characters (`"hello"`).

3. **Why 0-Based Indexing and `[start..end]` Fit Together?**

Because we start counting steps at `0`, the **length of a piece matches its right-side tick mark**.
* If you want a piece that is **5 characters long** starting from the beginning, you go from step `0` to step `5` (`[0..5]`).
* No math tricks, no adding or subtracting 1—the end index directly equals the total length!

4. **Why 1-Based Indexing Makes the Math Worse?**

If programming used 1-based indexing instead, you would have to constantly add or subtract `1` in your head:

* To get 5 characters starting at index 1, you would have to write `[1..5]`. But $5 - 1 = 4$, which doesn't equal 5! You'd have to write `[1..6]` or use inclusive ranges like `1..=5`.
* By starting at `0` and using exclusive end bounds (`[start..end]`), the math always remains simple subtraction: `Length = End - Start`.

---

## String Literals Are Slices (`&str`)

Whenever you write a hardcoded string inside quotes like `"hello world"`, you are creating a string slice.

```rust
let s = "Hello, world!"; // implicit type
let s: &str = "Hello, world!"; // explicit type
```





A **string literal** (like `"Hello, world!"`) is baked directly into your compiled executable binary file. When your program runs, it is loaded into a **read-only section of RAM**.

Because a string literal is already stored in memory, its Rust variable type is a string slice (`&str`)—a fat pointer on the **Stack** pointing directly into **Binary Data Memory**.

```text
       STACK MEMORY                                      READ-ONLY BINARY DATA (RAM)
+------------------------+                               +-------------------------+
| Address 1000: s (&str) |                               | Address | Character     |
|   ptr -----------------------------------------------> | 8000    | 'H'           |
|   len: 13              |                               | 8001    | 'e'           |
+------------------------+                               | 8002    | 'l'           |
                                                         | 8003    | 'l'           |
                                                         | 8004    | 'o'           |
                                                         | 8005    | ','           |
                                                         | 8006    | ' '           |
                                                         | 8007    | 'w'           |
                                                         | 8008    | 'o'           |
                                                         | 8009    | 'r'           |
                                                         | 8010    | 'l'           |
                                                         | 8011    | 'd'           |
                                                         | 8012    | '!'           |
                                                         +-------------------------+
```

* **Fat Pointer Structure:** The variable `s` on the stack takes up **16 bytes**—8 bytes for the starting address pointer (`8000`) and 8 bytes for the byte length (`13`).
* **No Heap Allocation:** Unlike `String::from("Hello, world!")`, string literals do not use heap memory or dynamic memory allocation.
* **Immutable by Default:** The operating system marks binary data segments as read-only. Attempting to modify a string literal directly would trigger a memory violation.
* **Static Lifetime (`'static`):** String literals remain available in memory for the entire duration of the program.

### Detailed Program Execution Flow

1. **Storage on Disk (Secondary Memory):**
    * Your compiled executable binary contains both the program's CPU instructions and hardcoded string literals in a read-only data section (often named `.rodata`).
2. **Loading into RAM (Main Memory):**
    * When you launch the executable, the Operating System's loader reads the file from the SSD/HDD and maps its segments into **RAM**.
3. **Runtime Operations:**
    * **String Literal (`&str`):** Points directly to the pre-loaded string inside the read-only region of RAM. No dynamic memory allocation takes place.
    * **Heap Allocation (`String`):** Calling `String::from("hello")` asks the OS memory allocator for a fresh block of bytes on the Heap in RAM, then copies the characters into that new dynamic allocation.

Modern Operating Systems give every running process a **Virtual Address Space** (usually 0 to 2⁶⁴-1 on 64-bit systems). Inside this virtual layout, memory is segregated into distinct, non-overlapping segments controlled by the OS page tables.

```text
  [ Low Addresses: ~0x00000000 ]
  +-----------------------------------+
  | .text (Executable CPU Code)       |  <-- Read-Only / Execute
  +-----------------------------------+
  | .rodata (String Literals, Consts) |  <-- Read-Only Data (Binary Data)
  +-----------------------------------+
  | .data / .bss (Global/Static Vars) |  <-- Read / Write
  +-----------------------------------+
  | HEAP (Grows UPWARD ↑)              |  <-- Read / Write (Dynamic Allocations)
  |      ↓                            |
  |      ... unmapped gap ...         |
  |      ↑                            |
  | STACK (Grows DOWNWARD ↓)          |  <-- Read / Write (Local Variables)
  +-----------------------------------+
  [ High Addresses: ~0x7FFFFFFF... ]

```

## How Stack, Heap, and Binary Data Function

* **Binary Data (`.rodata`):**
    * Placed near the **bottom (low memory addresses)** of the process's address space when the OS loads your file from disk.
    * Marked **Read-Only** at the hardware level by the CPU/MMU. If code attempts to write to a pointer targeting `.rodata`, the hardware triggers a Segmentation Fault immediately.
    * String literals (`&str`) store a pointer pointing directly down to this section.
* **The Stack:**
    * Located near the **top (high memory addresses)** and conventionally **grows downward** (towards lower memory addresses) as functions are called.
    * Uses a single register (the Stack Pointer) to reserve memory instantaneously by subtracting from its pointer value.
    * Holds local variables, fixed-size data, and pointers (`&String`, `&str`, raw addresses).
* **The Heap:**
    * Located **above the binary sections** and conventionally **grows upward** (towards higher memory addresses) as requested by `malloc` or Rust's allocator.
    * Used for dynamically sized data (`String`, `Vec<T>`).
    * Marked **Read-Write**, so its data can be mutated freely by whatever variable owns or borrows it with mutable permissions (`&mut`).

A string literal `&str` on the Stack holds an address to `.rodata` (read-only RAM), whereas a `String` on the Stack holds an address to the Heap (read-write RAM).

### Why the Architecture Work This Way

The structure of process memory isn't arbitrary, every region is placed and managed to optimize a specific set of hardware and software constraints.

```text
               STACK (Grows DOWN ↓)
               [Fast • Fixed-Size • LIFO]
                         │
                         ▼
                ... Free Space ...
                         ▲
                         │
                HEAP (Grows UP ↑)
               [Flexible • Dynamic • Allocator Overhead]
                         │
          ┌──────────────┴──────────────┐
          ▼                             ▼
   .rodata (Read-Only)          .text (Code Execution)
   [Hardware Enforced]         [Hardware Enforced]
```

#### The Stack

* **Why it's fast:** The Stack manages memory using a single CPU register (the Stack Pointer). "Allocating" space for local variables is literally just **a single subtraction instruction** on that pointer (moving it down). "Deallocating" when a function returns is just adding to the pointer (moving it back up).
* **Why it's at the high end:** Placing the Stack at one extreme end of the address space and letting it grow toward the middle allows it to expand dynamically without colliding with static program code.

#### The Heap

* **Why it's slower:** Unlike the Stack, Heap allocation requires finding an arbitrary chunk of available memory that fits your requested size. This involves a **Memory Allocator** (like `jemalloc` or system `malloc`) which must search its data structures, handle fragmentation, and track freed blocks.
* **Extra Operations:** If no suitable block is available, the allocator must make a system call to the OS kernel (`brk` or `mmap`) to expand the Heap's boundaries. Furthermore, accessing Heap memory requires **indirection**—loading the pointer address off the Stack first, then querying the target Heap address in RAM.
* **Why we use it:** It's the only way to support growable data structures (`Vec`, `String`) and data whose lifetime extends beyond the execution of a single function frame.

#### `.rodata` & `.text`

* **Hardware-Enforced Behavior:** During program startup, the OS tells the CPU’s Memory Management Unit (MMU) which permission bits apply to each address range:
    * `.text` (Machine instructions): **Read + Execute** (cannot modify code while running).
    * `.rodata` (String literals, constants): **Read-Only**.
    * Heap/Stack: **Read + Write** (Execution disabled via `NX/DEP` bits for security).

| Memory Region | Primary Trade-off | Access Speed | Size Flexibility | Management Overhead |
| --- | --- | --- | --- | --- |
| **Stack** | Maximum speed, zero flexibility | **Blazing fast** (Direct CPU register adjustment) | Fixed at compile time | Virtually zero |
| **Heap** | Maximum flexibility, lower speed | **Slower** (Pointer indirection + cache misses) | Dynamically growable | High (Allocator bookkeeping) |
| **`.rodata`** | Maximum safety, fixed contents | **Fast** (Pre-baked directly into binary) | Immutable / Read-only | Zero (Managed by OS loader) |

## The Three Main Target Locations for `&str`

A **`&str` (string slice)** is not tied to one specific location in memory. A slice is simply a **fat pointer** (a 16-byte structure on the Stack containing a starting address + a length). What changes depending on how the string was created is **where that starting address points**:


```text
1. String Literal ("hello")           2. Slice of Heap String               3. Slice of Stack Array
   STACK               BINARY DATA       STACK               HEAP              STACK
+----------+          +-----------+   +----------+          +-----------+   +----------+
| ptr: ----------------> "hello"  |   | ptr: ----------------> "hello"  |   | [u8; 5]: |
| len: 5   |          +-----------+   | len: 5   |          +-----------+   |  "hello" |
+----------+   (Read-Only RAM)        +----------+   (Read-Write RAM)       |          |
                                                                            | ptr: ----+ (points to stack!)
                                                                            | len: 5   |
                                                                            +----------+

```

#### 1. Binary Data Section (`.rodata`) — String Literals

When you write `let s: &str = "hello";`:

* **Stack:** Holds `ptr` and `len`.
* **Points to:** The read-only static memory loaded directly from your executable binary.
* **Lifetime:** `'static` (valid for the entire duration of the program).

#### 2. Heap Memory — Slice of a `String`

When you slice a heap-allocated `String`, e.g., `let s: &str = &my_string[0..5];`:

* **Stack:** Holds `ptr` and `len`.
* **Points to:** A specific byte offset inside the dynamic memory buffer allocated on the Heap.
* **Lifetime:** Tied to the borrowing scope of the parent `String` owner.

#### 3. Stack Memory — Slice of a Local Stack Array

You can even slice raw bytes on the Stack, e.g., 

```rust
let bytes: [u8; 5] = *b"hello";
let s: &str = std::str::from_utf8(&bytes).unwrap();
```

* **Stack:** Holds both the array data *and* the `&str` pointer.
* **Points to:** An address higher/lower on the Stack itself!

> [!NOTE]
> Think of `&str` as a **viewfinder**. It doesn't care *where* the underlying character bytes live—whether in read-only Binary Data, dynamic Heap memory, or local Stack memory. Its only job is to store the starting address and the length so the compiler knows what range of bytes you are allowed to read.

> [!NOTE]
> A slice (`&str` or `&[T]`) is **always just a fat pointer**—a 16-byte structure sitting on the stack consisting of a starting address pointer (`ptr`) and a byte/element count (`len`).

### The Distinction: Data vs. View

1. **The String Literal itself is the DATA sitting in RAM.**
* `"hello"` is a sequence of 5 UTF-8 bytes (`'h', 'e', 'l', 'l', 'o'`) stored in the read-only `.rodata` section of your executable.
* It is the raw content, not the pointer.


2. **The `&str` variable is the SLICE (the view into that data).**
* When you write `let s = "hello";`, `s` is a slice variable on your stack.
* Its `ptr` points to the starting address of `"hello"` inside `.rodata`, and its `len` is `5`.

> **"A string literal is read-only DATA in binary memory, and Rust gives you access to it through a String Slice (`&str`)."**

### Why All `&str` Slices Are Identical in Type

To Rust's type system, **all `&str` slices are structurally identical**. A `&str` pointing to `.rodata` works the exact same way as a `&str` pointing to the Heap or the Stack.

```rust
fn main() {
    // 1. Pointing to BINARY DATA (.rodata)
    let slice_binary: &str = "hello";

    // 2. Pointing to HEAP DATA
    let heap_string = String::from("hello world");
    let slice_heap: &str = &heap_string[0..5];

    // 3. Pointing to STACK DATA
    let stack_bytes: [u8; 5] = [104, 101, 108, 108, 111]; // ASCII for "hello"
    let slice_stack: &str = std::str::from_utf8(&stack_bytes).unwrap();

    // ALL THREE ARE THE EXACT SAME TYPE (&str)!
    // They are all just 16-byte fat pointers (ptr + len).
}
```

* **What is a Slice (`&str`)?** A 16-byte fat pointer (`ptr` + `len`) that views a sequence of contiguous string bytes.
* **Where can a Slice point?** Anywhere in memory—Binary Data (`.rodata`), Heap, or Stack.

## String Slices as Parameters (`&str`)

Now that you know a string slice (`&str`) can point to heap data (via `String`) OR binary executable data (via literals), Rust developers follow a gold-standard idiom:

> **Always prefer `&str` over `&String` when declaring function parameters!**

```rust
// ❌ RESTRICTIVE: Only accepts a full heap-allocated String reference
fn print_word(s: &String) {
    println!("{s}");
}

// ✅ IDIOMATIC & FLEXIBLE: Accepts String references AND String Literals!
fn print_word(s: &str) {
    println!("{s}");
}
```

## Deref Coercion (Automatic Slicing)

If you have a function that accepts `&str`, you can still pass a `&String` variable into it!

Rust performs a feature called **Deref Coercion**, which automatically converts a `&String` reference into a full string slice (`&s[..]`) behind the scenes without any performance penalty:

```rust,editable
fn print_word(s: &str) {
    println!("{s}");
}

fn main() {
    let my_string = String::from("hello world");
    let my_literal = "hello world";

    // Both work seamlessly with `fn print_word(s: &str)`!
    print_word(&my_string); // Automatically coerced from &String to &str (&my_string[..])
    print_word(my_literal);  // Already a &str
}
```

## Generic Slices: Array Slices (`&[T]`)

Slices are not exclusive to text! You can create a slice for **any contiguous collection in memory**, such as primitive arrays or vectors.

Syntax-wise, array slices operate identically to string slices by storing a starting address pointer and a length on the Stack:

```rust,editable
fn main() {
    let a: [i32; 5] = [10, 20, 30, 40, 50];

    // Create a slice borrowing elements at index 1 up to (not including) index 4
    let slice: &[i32] = &a[1..4]; // Refers to [20, 30, 40]

    assert_eq!(slice, &[20, 30, 40]);
}
```