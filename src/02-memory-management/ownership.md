# Ownership

Ownership enables Rust to guarantee memory safety without needing a garbage collector (like Python or Go) or requiring manual memory management (like C or C++).

## Stack vs. Heap

Imagine you are running a busy **restaurant kitchen**:

### The Stack: Your Kitchen Counter Top

Think of the Stack as a **small, ultra-fast organized stack of sticky notes** on your kitchen counter.

* **How it works:** You slap a note on top of the pile, and when you're done, you peel off the top note.
* **The Catch:** You *must* know exactly how big the information is before writing it down. You can't write a whole novel on a fixed 3x3 sticky note.
* **Why it's fast:** Everything is sitting right in front of you in a neat, fixed stack. The CPU knows *exactly* where every item is located without looking around.
* **Examples:** Simple numbers, booleans, fixed letters (`5`, `true`, `'a'`). They take up a tiny, predictable amount of memory space.

### The Heap: The Restaurant Storage Warehouse

Think of the Heap as a **huge, messy storage warehouse** down the street.

* **How it works:** When you order 500 lbs of flour, it won't fit on your sticky note. So, you call the warehouse manager. The manager finds a big empty spot in the warehouse, dumps the flour there, and gives you a **Claim Ticket** with an address written on it (e.g., *"Aisle 4, Shelf 12"*).
* **The Connection:** You stick that tiny **Claim Ticket on your counter sticky note** (on the Stack). Whenever you need flour, you look at the ticket address on your counter, walk down to the warehouse, and grab the flour.
* **Why it's slower:**
    1. Finding an open spot in a giant warehouse takes time (**Allocation**).
    2. Walking over to the warehouse address takes extra steps (**Following a Pointer**).
* **Examples:** Text strings or growing lists (`String`, `Vec`).

Look at what happens in memory when you create a `String`:

```rust
let name = String::from("Ferris");
```

```text
    STACK (Your Counter Top)                    HEAP (The Warehouse)
+-------------------------------+          +--------------------------+
|  variable: `name`             |          | Address | Value          |
|    Claim Ticket (Pointer) -------------> | 1004    | 'F'            |
|    Length: 6                  |          | 1005    | 'e'            |
|    Capacity: 6                |          | 1006    | 'r'            |
+-------------------------------+          | 1007    | 'r'            |
                                           | 1008    | 'i'            |
                                           | 1009    | 's'            |
                                           +--------------------------+
```

- If you throw away your **Claim Ticket** (the pointer on the stack), that flour is stuck in the warehouse forever, taking up space (**Memory Leak**).
- If two cooks try to use the same Claim Ticket and both try to throw it away when they're done, the warehouse gets confused and crashes (**Double Free Bug**).

> *"Only ONE cook gets to hold the Claim Ticket at any time. When that cook leaves the kitchen, they MUST throw away the item in the warehouse."*

|  | The Stack | The Heap |
| --- | --- | --- |
| **Analogy** | Sticky notes on your desk | A giant rental storage locker |
| **Size** | Small & strictly fixed size | Can grow as big as you need |
| **Speed** | ⚡ Blazing fast | 🐢 Slower (needs lookup time) |
| **What lives here?** | Small primitive data (`i32`, `bool`) AND **Claim Tickets (Pointers)** | Big or dynamic data (`String`, `Vec`) |

> [!NOTE]
> - **Stack**: Anything where the exact size in bytes is known at compile time.
> - **Heap**: Anything that can grow, shrink, or has a size unknown at compile time.

Ownership rules recap:

1. **Each value in Rust has an *owner*.**
2. **There can only be *one owner* at a time.**
3. **When the owner goes out of scope, the value will be dropped.**

## Variable Scope & The `drop` Function

A variable's **scope** is the range within a program for which the variable is valid.

```rust
fn main() {
    {                      // s is not valid here; it’s not yet declared
        let s = "hello";   // s is valid from this point forward

        // do stuff with s
    }                      // this scope is now over, and s is no longer valid!
}
```

When a variable that owns heap memory goes out of scope, Rust automatically calls a special function called **`drop`**.

* In C/C++, you must manually call `free(ptr)` or `delete`. Forget it, and you get a **memory leak**. Do it twice, and you get a **double-free bug**.
* In Rust, the compiler automatically inserts `drop` at the closing curly brace `}` of the owner's scope.

## Moves vs. Copies

How data is assigned or passed to functions depends on whether it lives on the Stack or Heap.

### Stack Data: Copying (`Copy` Trait)

Types that have a fixed size known at compile time are stored entirely on the stack. Copying these values is cheap and fast.

```rust,editable
let x = 5;
let y = x; // `x` is COPIED into `y`

println!("x = {x}, y = {y}"); // ✅ Valid! Both x and y exist independently.
```

```text
STACK (Your Desk)                      HEAP (Warehouse)
+-------------------------------+          +--------------------------+
|  x: 5                         |          |                          |
|  y: 5 (Independent Copy)      |          |   (EMPTY! Nothing on     |
+-------------------------------+          |    the heap at all)      |
                                           +--------------------------+
```

### Heap Data: The "Move"

For heap data (like `String`), assignment **does not perform a deep copy** of the heap data because that would be expensive for performance.

```rust,editable
let s1 = String::from("hello");
let s2 = s1; // ⚠️ Data is MOVED from s1 to s2!
```

1. You order "hello" in the heap warehouse. You get a claim ticket (`s1`) on your stack desk pointing to memory address `1004`.

```text
       STACK (Your Desk)                      HEAP (Warehouse)
+-------------------------------+          +--------------------------+
|  s1                           |          | Address | Value          |
|    ptr --------------------------------> | 1004    | 'h'            |
|    len: 5                     |          | 1005    | 'e'            |
|    cap: 5                     |          | 1006    | 'l'            |
+-------------------------------+          | 1007    | 'l'            |
                                           | 1008    | 'o'            |
                                           +--------------------------+
```

2. Rust copies the tiny **stack ticket** (ptr, len, cap) into `s2`. **BUT** Rust immediately marks `s1` as **INVALID** (or uninitialized) in its tracking compiler ledger.

```text
STACK (Your Desk)                       HEAP (Warehouse)
+-------------------------------+
|  s1  [INVALIDATED / MOVED]    |
|    ptr --------------------X  | (Dead)
|    len: 5                     |
|    cap: 5                     |
+-------------------------------+          +---------+---------------+
|  s2                           |          | Address | Value ('hello')|
|    ptr --------------------------------> |  1004   | 'h'           |
|    len: 5                     |          |  1005   | 'e'           |
|    cap: 5                     |          |  1006   | 'l'           |
+-------------------------------+          |  1007   | 'l'           |
                                           |  1008   | 'o'           |
                                           +---------+---------------+
```

The stack memory slot for `s1` physically remains in RAM until the function exits, but **the compiler strictly forbids you from reading or using `s1` ever again**.

```rust,editable
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{s1}"); // ❌ COMPILE ERROR: borrow of moved value: `s1`
}
```

This operation is called a **Move**. Ownership of the heap data was moved from `s1` to `s2`, making `s2` the sole owner.

If both `s1` and `s2` remained valid, both would point to the same memory block (`1004`). When the function ends and goes out of scope:

1. `s1` would try to free the memory at `1004`.
2. `s2` would try to free the memory at `1004` **a second time**.
3. 💥 **Double-Free Bug:** Trying to free memory that was already freed leads to memory corruption and security vulnerabilities.

Because Rust invalidates `s1` during the Move:

* When going out of scope, Rust **ignores `s1` entirely** (it calls `drop()` on valid owners only).
* Rust calls `drop()` **once** on `s2`, safely freeing address `1004`.

## Reassigning a Mutable `String`

When you reassign a mutable variable that owns dynamic heap data, Rust **immediately drops the old heap memory** before taking ownership of the new value.

```rust,editable
fn main() {
    let mut s = String::from("hello");
    s = String::from("world"); // ⚠️ Reassignment drops "hello" immediately!
}
```

1. **Initial Allocation:** `s` is created on the stack pointing to Address `1004` (`"hello"`).

```text
       STACK (Your Desk)                      HEAP (Warehouse)
+-------------------------------+          +--------------------------+
|  s                            |          | Address | Value          |
|    ptr --------------------------------> | 1004    | 'h'            |
|    len: 5                     |          | 1005    | 'e'            |
|    cap: 5                     |          | 1006    | 'l'            |
+-------------------------------+          | 1007    | 'l'            |
                                           | 1008    | 'o'            |
                                           +--------------------------+
```

2. **Reassignment Trigger:** `String::from("world")` allocates new heap space at Address `2050`.
3. **Automatic Intermediate `drop`:** Before pointing `s` to the new address, Rust automatically calls `drop()` on Address `1004` (`"hello"`). The original memory is **freed immediately on that line**—you do not have to wait until the closing brace `}`.
4. **New Ownership:** The stack frame for `s` is updated to point to Address `2050` (`"world"`).

```text
        STACK (Your Desk)                        HEAP (Warehouse)
                                           +---------+-------------------+
                                           | Address | Value             |
                                           +---------+-------------------+
                                           |  1004   | [FREED / DROPPED] |
+-------------------------------+          |  ...    |                   |
|  s                            |          +---------+-------------------+
|    ptr --------------------------------> |  2050   | 'w'               |
|    len: 5                     |          |  2051   | 'o'               |
|    cap: 5                     |          |  2052   | 'r'               |
+-------------------------------+          |  2053   | 'l'               |
                                           |  2054   | 'd'               |
                                           +---------+-------------------+
```

5. **Scope Exit (`}`):** When `s` eventually goes out of scope, Rust calls `drop()` **once** on `s`, freeing Address `2050` (`"world"`).

## Cloning (Deep Copy)

If you *do* want a deep copy of heap data (duplicating both stack pointer and heap memory), you must explicitly call the `.clone()` method.

```rust,editable
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone(); // Allocates new memory on the heap and copies data

    println!("s1 = {s1}, s2 = {s2}"); // ✅ Valid! Both own separate heap allocations.
}
```

Because both variables now own completely independent heap memory addresses, **neither variable is invalidated**—both `s1` and `s2` remain fully valid!

You allocate memory on the heap for `"hello"`. `s1` owns address `1004`.

```text
       STACK (Your Desk)                      HEAP (Warehouse)
+-------------------------------+          +--------------------------+
|  s1                           |          | Address | Value          |
|    ptr --------------------------------> | 1004    | 'h'            |
|    len: 5                     |          | 1005    | 'e'            |
|    cap: 5                     |          | 1006    | 'l'            |
+-------------------------------+          | 1007    | 'l'            |
                                           | 1008    | 'o'            |
                                           +--------------------------+
```

Rust sees `.clone()` and performs two distinct steps:

1. Finds a **new, open spot on the heap** (e.g., starting at address `2050`) and duplicates the text `'h','e','l','l','o'`.
2. Creates a **new stack ticket `s2`** pointing directly to that *new* address `2050`.

```text
        STACK (Your Desk)                       HEAP (Warehouse)
+-------------------------------+          +---------+------------------+
|  s1                           |          | Address | Value            |
|    ptr --------------------------------> |  1004   | 'h'              |
|    len: 5                     |          |  1005   | 'e'              |
|    cap: 5                     |          |  1006   | 'l'              |
+-------------------------------+          |  1007   | 'l'              |
|  s2                           |          |  1008   | 'o'              |
|    len: 5                     |          +---------+------------------+
|    ptr --------------------------------> |  2050   | 'h' (Cloned)     |
|    cap: 5                     |          |  2051   | 'e'              |
+-------------------------------+          |  2052   | 'l'              |
                                           |  2053   | 'l'              |
                                           |  2054   | 'o'              |
                                           +---------+------------------+
```

> [!NOTE]
> **What happens when going out of scope `}`?**
> - When the function block finishes:
>   1. Rust calls `drop()` on `s1` ➡️ Frees Heap Address `1004`.
>   2. Rust calls `drop()` on `s2` ➡️ Frees Heap Address `2050`.
> - **Result:** Zero double-free bugs! Since they point to entirely different locations in memory, both clean up their own heap space safely.

| Operation | Heap Data Duplicated? | Stack Ticket Copied? | Old Variable (`s1`) Status | Performance Cost |
| --- | --- | --- | --- | --- |
| **Move** (`let s2 = s1;`) | ❌ No | ✅ Yes (24 bytes) | 🪦 **Invalidated** | ⚡ Blazing fast |
| **Clone** (`let s2 = s1.clone();`) | ✅ Yes (Full copy) | ✅ Yes (24 bytes) | ✅ **Stays Alive** | 🐢 Slower (Heap allocation) |

## Ownership in Functions

Passing a variable to a function behaves just like an assignment: it will either **move** or **copy** the value.

```rust,editable
fn main() {
    let s = String::from("hello"); // s comes into scope

    takes_ownership(s);             // s's value MOVES into the function...
                                   // ... and is no longer valid here!

    let x = 5;                     // x comes into scope

    makes_copy(x);                 // x MOVES into function, but i32 is Copy,
                                   // so x is still valid here!

} // Here, x goes out of scope, then s. But s's value was moved, so nothing happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{some_string}");
} // some_string goes out of scope and `drop` is called. Memory freed!

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{some_integer}");
} // some_integer goes out of scope. Nothing special happens.
```

### Is `drop` called on `some_integer`?

**No, `drop` is NOT called for `some_integer` or `x`!**

The `drop()` function is specifically designed to clean up **heap memory allocations** (like releasing dynamic memory back to the operating system).

Because `i32` integers live **entirely on the Stack**, there is no heap memory to free!

* **Heap types (`String`, `Vec`):** Implement the `Drop` trait. When they go out of scope, Rust explicitly runs `drop()` to deallocate their heap memory.
* **Primitive Stack types (`i32`, `f64`, `bool`):** Implement the `Copy` trait instead. They **do not implement `Drop`**. When they go out of scope, Rust simply moves the CPU stack pointer back (popping the stack frame). It is instantaneous and requires zero cleanup functions.

```rust
fn main() {
    let s = String::from("hello"); 

    takes_ownership(s); // 1. s MOVES into `takes_ownership`
                        //    `s` on main's stack is now DEAD/INVALID.

    let x = 5;          
    makes_copy(x);      // 2. A COPY of x (5) is passed to `makes_copy`.
                        //    `x` inside main remains 100% ALIVE.

} // 3. `main()` ends! What happens to `x` and `s` here?
```

1. **Inside `takes_ownership(some_string: String)`:**
When `takes_ownership` finishes, `some_string` goes out of scope. **This is where `drop()` is called** to free the `"hello"` heap memory right away!
2. **Inside `makes_copy(some_integer: i32)`:**
When `makes_copy` finishes, `some_integer` goes out of scope. Its stack slot is popped off. No `drop()` needed.
3. **At the end of `main()` (`}`):**
    * **For `s`:** It went out of scope, but because its value was **already moved** into `takes_ownership` (where it was already freed), Rust does **nothing**.
    * **For `x`:** It goes out of scope. Its stack memory slot in `main` is popped off. No `drop()` needed.

| Type Category | Trait Implemented | On Move/Assignment | Scope Cleanup (`}`) |
| --- | --- | --- | --- |
| **Heap Data** (`String`, `Vec`) | `Drop` | **Move** (Original invalidated) | Calls `drop()` to free Heap memory |
| **Stack Data** (`i32`, `bool`, `char`) | `Copy` | **Copy** (Original remains valid) | Stack frame pops (No `drop()` call) |

## What is a "Stack Frame"?

A **Stack Frame** (also called a call frame) is a dedicated, contiguous block of memory allocated on top of the Stack every time a function is invoked.

It acts as an isolated workspace for that specific function call, holding everything the CPU needs to execute the function and safely return when finished.

```text
                  STACK MEMORY
+-----------------------------------------------+
|  [ Caller Function Frame ]                    |
|    Local Variables & State of Calling Function|
+-----------------------------------------------+
|  [ Called Function Frame ]                    |
|    • Return Address (where to resume)         |
|    • Function Parameters                      |
|    • Local Variables                          |
|    • Frame Bookkeeping (saved registers)      |
+-----------------------------------------------+ <-- Stack Pointer (SP)
```

### What Lives Inside a Stack Frame?

Every stack frame contains four core items, laid out in sequential memory addresses:

* **Return Address:** The CPU memory address pointing to the exact instruction line in the caller function. When the function ends, the CPU reads this address to know where to resume execution.
* **Function Parameters:** The values or references passed into the function arguments.
* **Local Variables:** All variables declared inside the function body (e.g., primitive values or stack headers for `String` and `&str`).
* **Frame Bookkeeping:** Saved register states (like the previous Frame Pointer) that allow the CPU to restore the caller function's exact environment upon return.

### Key Properties

* **Isolated Lifetime:** A stack frame exists **only while its function is running**. As soon as the function returns, its frame is no longer active.
* **LIFO Structure (Last-In, First-Out):** If function `A` calls function `B`, and `B` calls function `C`, their frames stack on top of each other (`A ➡️ B ➡️ C`). `C` must finish and destroy its frame before `B` can finish, which must finish before `A` can finish.
* **Predictable Size:** The compiler calculates the exact byte size of a function's stack frame at compile time by adding up the sizes of all its parameters and local variables.

### What is "Popping the Stack Frame"?

Inside your computer's CPU, a dedicated register called the **Stack Pointer (`SP`)** tracks the top edge of active stack memory.

When a function is called, the CPU **pushes** a new Stack Frame onto the stack. When the function hits its closing brace `}`, the CPU **pops** that frame off by adjusting the Stack Pointer back to where it was before the call.

"Popping" does not mean erasing memory, running cleanup code, or setting bytes to zero. It literally means **adjusting a single pointer register in the CPU**. The memory becomes available space simply because the CPU no longer considers those addresses part of an active frame.

```rust
fn main() {
    let a: i32 = 10;
    makes_copy(a); // <--- Function call!
    let b: i32 = 20;
} 

fn makes_copy(some_integer: i32) {
    let internal_var: i32 = 99;
    println!("{some_integer}");
}
```

### Step-by-Step Memory Execution Trace

#### Step 1: Active `main()` Frame Before Call

The Stack Pointer (`SP`) sits at Address `1004`, right after `a`.

```text
       STACK MEMORY (RAM)
+------------------------------------+
| Address   | Component              |
+-----------+------------------------+
| 1000..1003| main Frame: a = 10     |
+------------------------------------+ <-- STACK POINTER (SP = 1004)
| (Unused Memory Space)              |
+------------------------------------+
```

#### Step 2: Pushing `makes_copy` Stack Frame

When `makes_copy(a)` is invoked, the CPU allocates a 24-byte stack frame containing all four standard frame components. `SP` moves down to `1028`.

```text
       STACK MEMORY (RAM)
+-------------------------------------------------------------------+
| Address   | Frame Component          | Value / Purpose            |
+-----------+--------------------------+----------------------------+
| 1000..1003| main Frame               | a: i32 = 10                |
+-----------+--------------------------+----------------------------+
| 1004..1011| 1. Return Address        | Address of "let b = 20;"   |
| 1012..1019| 2. Frame Bookkeeping     | Saved RBP (main's base ptr)|
| 1020..1023| 3. Parameter             | some_integer: i32 = 10     |
| 1024..1027| 4. Local Variable        | internal_var: i32 = 99     |
+-------------------------------------------------------------------+ <-- STACK POINTER (SP = 1028)
| (Unused Memory Space)                                             |
+-------------------------------------------------------------------+
```

#### Step 3: Popping the Frame (Function Exit at `}`)

When `makes_copy` completes:

1. The CPU reads **Return Address** (`1004..1011`) to resume execution at `let b = 20;`.
2. The CPU reads **Frame Bookkeeping** (`1012..1019`) to restore `main`'s frame base pointer.
3. The CPU adjusts **`SP` back to `1004**`.

```text
       STACK MEMORY (RAM)
+-------------------------------------------------------------------+
| Address   | State / Contents         | Note                       |
+-----------+--------------------------+----------------------------+
| 1000..1003| main Frame: a = 10       | Active Memory              |
+-------------------------------------------------------------------+ <-- STACK POINTER (SP = 1004)
| 1004..1011| [Return Address]         | ┐                          |
| 1012..1019| [Saved RBP]              | │ GHOST DATA IN RAM        |
| 1020..1023| [some_integer = 10]      | │ (Bits sit unchanged,     |
| 1024..1027| [internal_var = 99]      | ┘  but ignored by CPU)     |
+-------------------------------------------------------------------+
```

#### Step 4: Reusing Memory (`let b: i32 = 20;`)

`main()` resumes and creates `b`. Because `SP` is at `1004`, `b` is placed at addresses `1004..1007`, directly overwriting part of the abandoned frame.

```text
       STACK MEMORY (RAM)
+-------------------------------------------------------------------+
| Address   | Variable / Contents      | Note                       |
+-----------+--------------------------+----------------------------+
| 1000..1003| main Frame: a = 10       | Active Memory              |
| 1004..1007| main Frame: b = 20       | Overwrote Return Address!  |
+-------------------------------------------------------------------+ <-- STACK POINTER (SP = 1008)
| 1008..1019| [Old Saved RBP Bytes]    | ┐ Remaining ghost data     |
| 1020..1027| [Old Variable Bytes]     | ┘ waiting to be overwritten|
+-------------------------------------------------------------------+
```

| Memory Location | How Cleanup Is Handled | CPU Operations Required |
| --- | --- | --- |
| **Stack Frame** | **Stack Pointer Pop:** CPU moves register pointer back to caller frame base. | **1 Instruction** (`ret` / instantaneous) |
| **Heap Memory** | **`Drop` Execution:** Allocator looks up pointer, updates free-block tables, releases RAM to OS. | **Multiple Operations** (malloc/free bookkeeping) |


### How is the Return Address known before reaching `let b = 20`?

Before your Rust program ever runs, the **compiler** translates all your code into binary machine instructions and lays them out sequentially in a separate section of your computer's RAM called **Code Memory** (or Text Segment).

Every line of executable machine code gets a fixed, permanent memory address before execution even starts.

```text
                  CODE MEMORY (Compiled Machine Instructions)
Memory Address | Instruction
---------------+--------------------------------------------------------------
  0x2000       | Address for:  let a = 10;
  0x2008       | Address for:  makes_copy(a);   <-- CPU is currently executing this!
  0x2016       | Address for:  let b = 20;      <-- The VERY NEXT instruction in memory!
```

When the CPU executes the instruction at `0x2008` (`makes_copy(a)`):

1. The CPU looks at the **next instruction line right after it** in Code Memory, which is `0x2016` (`let b = 20`).
2. It takes that exact number (`0x2016`) and pushes it onto the stack frame as the **Return Address**.
3. It then jumps away to execute `makes_copy`.

The line `let b = 20` doesn't need to be *executed* yet for the CPU to know *where* it lives in memory. The compiler already mapped out its location when creating the binary file.

### What is "Frame Bookkeeping"? (Simplified)

As a beginner, you can think of Frame Bookkeeping simply as a **saved bookmark pointing to the start of the previous function's memory**.

#### Why does the CPU need this bookmark?

The CPU uses a ruler (a register called the **Frame Pointer**) to find a function's local variables relative to the start of its memory frame.

```text
               STACK MEMORY
+------------------------------------------+  <-- Main's Frame Pointer ("Ruler" is pinned here)
|  a: i32 = 10                             |
+------------------------------------------+
```

When `main()` calls `makes_copy()`, the CPU must move that "ruler" down to start tracking `makes_copy()`'s variables.

If the CPU just moves the ruler without saving where it was originally pinned, **it will permanently forget where `main()`'s variables started!**

```text
               STACK MEMORY
+------------------------------------------+  
|  a: i32 = 10                             |  <-- Main's start location is LOST!
+------------------------------------------+  
|  Return Address: 0x2016                  |
|  [Saved Bookmark]                        |  <-- "Main's ruler was at Address 1000"
|  some_integer = 10                       |
|  internal_var = 99                       |
+------------------------------------------+  <-- Makes_copy's Frame Pointer ("Ruler" moved here)
```

#### What Frame Bookkeeping actually does:

1. **Before moving the ruler:** `makes_copy` saves a note on its own stack frame: *"The ruler for `main()` was at Address 1000."*
2. **When returning:** `makes_copy` reads that note and resets the ruler back to Address `1000`. Now `main()` has its environment restored seamlessly.

That saved note—the "saved frame pointer"—is what low-level documentation calls **Frame Bookkeeping**.

## Return Values and Scope

Returning a value from a function transfers ownership of that value back to the calling function, preventing Rust from calling `drop()` on the returned heap data.

```rust,editable
fn main() {
    let s1 = gives_ownership();        // gives_ownership MOVES its return
                                       // value into s1

    let s2 = String::from("hello");    // s2 comes into scope

    let s3 = takes_and_gives_back(s2); // s2 MOVES into takes_and_gives_back,
                                       // which MOVES its return value into s3
} 
// Here:
// 1. s3 goes out of scope and drop() frees its heap memory.
// 2. s2 went out of scope, but was MOVED, so nothing happens.
// 3. s1 goes out of scope and drop() frees its heap memory.

fn gives_ownership() -> String {
    let some_string = String::from("yours"); // some_string comes into scope
    some_string                              // some_string is RETURNED and
                                             // MOVES out to the caller!
}

fn takes_and_gives_back(a_string: String) -> String { // a_string comes into scope
    a_string                                          // a_string is RETURNED and
                                                      // MOVES out to the caller!
}
```

When `gives_ownership()` creates a `String` and returns it, ownership is passed up to `s1` in `main()` without deallocating the heap memory:

1. **Inside `gives_ownership()`:** `some_string` is created on the heap at Address `3010`.

```text
       STACK (Your Desk)                      HEAP (Warehouse)
+-------------------------------+          +--------------------------+
|  [gives_ownership Frame]      |          | Address | Value          |
|  some_string                  |          | 3010    | 'y'            |
|    ptr --------------------------------> | 3011    | 'o'            |
|    len: 5                     |          | 3012    | 'u'            |
|    cap: 5                     |          | 3013    | 'r'            |
+-------------------------------+          | 3014    | 's'            |
                                           +--------------------------+

```

2. **Returning to `main()`:** Ownership moves to `s1`. The stack ticket (ptr, len, cap) is copied into `main`'s stack frame pointing to Address `3010`.

```text
       STACK (Your Desk)                      HEAP (Warehouse)
+-------------------------------+          +--------------------------+
|  [main Frame]                 |          | Address | Value          |
|  s1                           |          | 3010    | 'y'            |
|    ptr --------------------------------> | 3011    | 'o'            |
|    len: 5                     |          | 3012    | 'u'            |
|    cap: 5                     |          | 3013    | 'r'            |
+-------------------------------+          | 3014    | 's'            |
|  [gives_ownership Frame]      |          +--------------------------+
|  some_string [INVALIDATED]    |
|    ptr ----------------------XX  (Dead)
+-------------------------------+

```

3. **Function Scope Exit (`}`):** The `gives_ownership` stack frame pops. Because `some_string` was moved out, **`drop()` is NOT called** on Address `3010`. The heap allocation stays alive, now owned exclusively by `s1` in `main()`.

| Movement Pattern | What Happens to Ownership? | Heap Data Status |
| --- | --- | --- |
| **Assigning to variable** (`let y = x;`) | Moves or Copies value | Intact (Owner changes) |
| **Passing into function** (`func(x)`) | Moves or Copies value into function parameter | Intact (New function scope owns it) |
| **Returning from function** (`return x;`) | Moves value out to caller binding | Intact (Caller scope takes ownership) |

> [!NOTE]
> **Key Takeaway:** Ownership always follows the value. If a function receives or creates heap data, that data will either be **dropped** at the function's closing brace `}` OR **moved out** via a return value or assignment.