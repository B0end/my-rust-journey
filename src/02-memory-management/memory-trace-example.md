# Memory Trace Example

To tie all of this memory theory together, here is a complete step-by-step memory trace of a Rust program. It illustrates how the **Stack**, **Heap**, and **Read-Only Binary Data (`.rodata`)** interact during variable creation, string literals, copying, moving, and slicing.

```rust
const MAX_ITEMS: u32 = 100;

fn main() {
    // 1. Primitive Copy (Stack-only)
    let x: i32 = 42;
    let y = x; 

    // 2. String Literal (&str -> .rodata)
    let lit: &str = "hello";

    // 3. Dynamic Heap Allocation (String)
    let mut s1 = String::from("rust");

    // 4. Move Semantics (Ownership Transfer)
    let s2 = s1; 

    // 5. Slicing Heap Data (&str -> Heap)
    let slice: &str = &s2[0..2];
}
```

## Step 0
### Program Compilation & Load Time

* **`const MAX_ITEMS`**: Has **no memory address**. The compiler performs "inlining"—it directly replaces every occurrence of `MAX_ITEMS` with the value `100` in the machine instructions (`.text`).
* **`"hello"`**: Hardcoded into the **Read-Only Binary Segment (`.rodata`)** at RAM Address `8000` when the OS loads the binary.

```text
READ-ONLY BINARY DATA (.rodata)
+-----------------------+
| Address | Byte Value  |
+---------+-------------+
| 8000    | 'h'         |
| 8001    | 'e'         |
| 8002    | 'l'         |
| 8003    | 'l'         |
| 8004    | 'o'         |
+-----------------------+
```

## Step 1
### Primitive Copy (`let x = 42; let y = x;`)

Primitives with a fixed, known size (like `i32`) implement the `Copy` trait. Pushing `x` onto the Stack creates a direct bitwise copy for `y`. Both exist independently on the Stack.

> [!NOTE]
> A bitwise copy means the CPU executes a direct memory copy instruction (`MOV` or `memcpy`) to duplicate the raw binary sequence of 0s and 1s from one memory address to another.

```text
STACK MEMORY (RAM)
+------------------------------------+
| Address   | Variable / Value       |
+-----------+------------------------+
| 1000..1003| x: i32 = 42            |
| 1004..1007| y: i32 = 42            |  <-- Bitwise copy of x (4 bytes)
+------------------------------------+
```

## Step 2
### String Literal (`let lit = "hello";`)

`lit` is a fat pointer (`ptr` + `len`) pushed onto the Stack. Its pointer points directly to address `8000` in `.rodata`.

```text
STACK MEMORY                             READ-ONLY BINARY DATA (.rodata)
+------------------------------------+   +-----------------------+
| Address   | Variable / Value       |   | Address | Byte Value  |
+-----------+------------------------+   +---------+-------------+
| 1000..1003| x: 42                  |   | 8000    | 'h'         |
| 1004..1007| y: 42                  |   | 8001    | 'e'         |
| 1008..1015| lit.ptr = 8000 --------+-->| 8002    | 'l'         |
| 1016..1023| lit.len = 5            |   | 8003    | 'l'         |
+------------------------------------+   | 8004    | 'o'         |
                                         +-----------------------+
```

## Step 3
### Dynamic Heap Allocation (`let mut s1 = String::from("rust");`)

Calling `String::from` requests memory from the Heap allocator. The Stack holds `s1`'s metadata (`ptr`, `len`, `cap`), which points to the dynamic heap address `5000`.

```text
STACK MEMORY                             HEAP MEMORY (Read/Write RAM)
+------------------------------------+   +-----------------------+
| 1008..1023| lit (&str)             |   | Address | Byte Value  |
+-----------+------------------------+   +---------+-------------+
| 1024..1031| s1.ptr = 5000 ---------+-->| 5000    | 'r'         |
| 1032..1039| s1.len = 4             |   | 5001    | 'u'         |
| 1040..1047| s1.cap = 4             |   | 5002    | 's'         |
+------------------------------------+   | 5003    | 't'         |
                                         +-----------------------+
```

## Step 4
### Move Semantics (`let s2 = s1;`)

Because `String` owns heap memory, assigning `s1` to `s2` performs a **Move** (not a copy).

1. The 24-byte header (`ptr`, `len`, `cap`) is copied on the Stack from `s1` to `s2`.
2. `s1` is marked **uninitialized/invalid** by the compiler to prevent double-free errors.
3. No heap data is duplicated.

```text
STACK MEMORY                             HEAP MEMORY (Read/Write RAM)
+------------------------------------+   +-----------------------+
| 1024..1047| s1 [INVALIDATED]       |   | Address | Byte Value  |
| 1048..1055| s2.ptr = 5000 ---------+-->| 5000    | 'r'         |
| 1056..1063| s2.len = 4             |   | 5001    | 'u'         |
| 1064..1071| s2.cap = 4             |   | 5002    | 's'         |
+------------------------------------+   | 5003    | 't'         |
                                         +-----------------------+
```

## Step 5
### Slicing Heap Data (`let slice = &s2[0..2];`)

Creating `slice` (`&s2[0..2]`) constructs a 16-byte fat pointer on the Stack. It bypasses `s2`'s stack address and points **directly to Heap Address 5000** with a length of `2`.

```text
STACK MEMORY                             HEAP MEMORY (Read/Write RAM)
+------------------------------------+   +-----------------------+
| 1048..1071| s2 (String owner)      |   | Address | Byte Value  |
+-----------+------------------------+   +---------+-------------+
| 1072..1079| slice.ptr = 5000 ------+-->| 5000    | 'r'  ┐      |
| 1080..1087| slice.len = 2          |   | 5001    | 'u'  ┴ &str |
+------------------------------------+   | 5002    | 's'         |
                                         | 5003    | 't'         |
                                         +-----------------------+
```

## Memory Operation Summary

| Operation | Stack Action | Heap Action | `.rodata` Action |
| --- | --- | --- | --- |
| **`const`** | None (Inlined into CPU instructions) | None | None |
| **Primitive (`i32`, `bool`)** | Stores fixed value directly | None | None |
| **String Literal (`&str`)** | Stores 16-byte Fat Pointer (`ptr` + `len`) | None | Points to pre-loaded binary data |
| **`String::from(...)`** | Stores 24-byte Owner (`ptr` + `len` + `cap`) | Allocates dynamic array buffer | None (or copies initial bytes) |
| **Primitive Copy** | Bitwise duplicate on Stack | None | None |
| **Resource Move** | Copies Stack header; invalidates old variable | Heap ownership transferred (0 reallocations) | None |
| **Slicing (`&str`)** | Stores 16-byte Fat Pointer (`ptr` + `len`) | Points directly to offset inside Heap buffer | None |

## Reallocation (Heap Growth)

When a growable structure like `String` needs to hold more bytes than its current `capacity`, the memory allocator must allocate a **new, larger contiguous chunk** elsewhere in Heap RAM, copy the existing data over, write the new bytes, and free the old chunk.

```rust
s2.push_str("ing"); // Extends "rust" (4 bytes) to "rusting" (7 bytes)
```

1. **Capacity Check:** `s2` has `len: 4` and `cap: 4`. Adding 3 bytes requires 7 bytes, exceeding `cap`.
2. **New Allocation:** The allocator searches Heap RAM for a larger free block (typically doubling capacity to 8 bytes) and allocates Address range `6000..6007`.
3. **Data Copy:** The CPU copies the original 4 bytes (`'r','u','s','t'`) from `5000..5003` to `6000..6003`.
4. **Append New Bytes:** The new bytes (`'i','n','g'`) are written to `6004..6006`.
5. **Old Block Deallocation:** Heap Address range `5000..5003` is freed back to the OS memory pool.
6. **Stack Metadata Update:** `s2`'s `ptr` is updated to `6000`, `len` becomes `7`, and `cap` becomes `8`.

```text
BEFORE REALLOCATION:
STACK MEMORY                             HEAP MEMORY (Read/Write RAM)
+------------------------------------+   +-----------------------+
| 1048..1055| s2.ptr = 5000 ---------+-->| 5000    | 'r'         |
| 1056..1063| s2.len = 4             |   | 5001    | 'u'         |
| 1064..1071| s2.cap = 4             |   | 5002    | 's'         |
+------------------------------------+   | 5003    | 't'         |
                                         +-----------------------+

---------------------------------------------------------------------

AFTER s2.push_str("ing"):
STACK MEMORY                             HEAP MEMORY (Read/Write RAM)
+------------------------------------+   +-----------------------+
| 1048..1055| s2.ptr = 6000 -----+   |   | 5000..5003 [FREED]    |
| 1056..1063| s2.len = 7         |   |   +-----------------------+
| 1064..1071| s2.cap = 8         |   +-->| 6000    | 'r'         |
+------------------------------------+   | 6001    | 'u'         |
                                         | 6002    | 's'         |
                                         | 6003    | 't'         |
                                         | 6004    | 'i'         |
                                         | 6005    | 'n'         |
                                         | 6006    | 'g'         |
                                         | 6007    | (Unused)    |
                                         +-----------------------+
```

## Drop / Deallocation (RAII Scope Cleanup)

When a variable that owns heap memory goes out of scope (at the closing brace `}` of a function or block), Rust automatically executes its `Drop` implementation. This prevents memory leaks without needing a garbage collector.

```rust
fn main() {
    let s2 = String::from("rusting");
    // ...
} // <-- s2 goes out of scope here
```

1. **Drop Execution:** The compiler automatically inserts `Drop::drop(&mut s2)` right before the scope closes.
2. **Heap Deallocation:** `String`'s drop function issues a free request to the allocator using `s2.ptr` (`6000`) and `s2.cap` (`8`).
3. **Memory Marked Free:** Address range `6000..6007` in Heap RAM is marked as available for future allocations.
4. **Non-Owner Cleanup:** Slices (`&str`) do not implement `Drop` because they don't own the underlying data—only owners free heap memory.

```text
BEFORE DROP:
STACK MEMORY                             HEAP MEMORY (Read/Write RAM)
+------------------------------------+   +-----------------------+
| 1048..1055| s2.ptr = 6000 ---------+-->| 6000..6007 | "rusting"|
| 1056..1063| s2.len = 7             |   +-----------------------+
| 1064..1071| s2.cap = 8             |
+------------------------------------+

---------------------------------------------------------------------

AFTER DROP (s2 scope ends):
STACK MEMORY                             HEAP MEMORY (Read/Write RAM)
+------------------------------------+   +-----------------------+
| 1048..1071| s2 [DROPPED]           |   | 6000..6007 [FREED]    |
+------------------------------------+   +-----------------------+
                                           (Allocated heap memory 
                                            returned to system)
```

## Stack Frame Teardown (Zero-Cost Function Exit)

When a function completes, the CPU destroys the entire local Stack Frame instantaneously. Unlike the Heap, individual stack memory addresses do not need to be manually erased or freed.

```rust
} // End of main() function
```

1. **Stack Pointer Register (`RSP`):** The CPU uses a dedicated hardware register called the Stack Pointer (`RSP`) to track the current top of the Stack.
2. **Pointer Adjustment:** Upon executing the function's return instruction (`ret`), the CPU adjusts `RSP` back to the caller's stack address (e.g., moving `RSP` from `1087` back to `1000` or higher).
3. **Zero Runtime Cost:** The memory slots (`1000..1087`) are not zeroed out or overwritten—they are simply marked as unallocated stack space. Future function calls will write directly over these addresses.

```text
ACTIVE STACK FRAME (During main execution):
STACK MEMORY (RAM)
+------------------------------------+ 
| Address   | Variable               | 
+-----------+------------------------+ 
| 1000..1007| x, y (i32)             | 
| 1008..1023| lit (&str)             | 
| 1024..1071| s1, s2 (String owners) | 
| 1072..1087| slice (&str)           | <-- Stack Pointer (RSP = 1087)
+------------------------------------+

---------------------------------------------------------------------

AFTER FUNCTION EXIT (Stack Frame Teardown):
STACK MEMORY (RAM)
+------------------------------------+ <-- Stack Pointer (RSP restored to <= 1000)
| 1000..1087| [INVALIDATED STACK]    | 
|           | (Data remains in RAM   | 
|           |  until overwritten by  | 
|           |  the next function)    | 
+------------------------------------+

```