# Smart Pointers

Pointers are variables that store a memory address. The most common pointers you've seen so far are **references** (`&`), which borrow data and have no special overhead.

**Smart Pointers**, on the other hand, are data structures that act like pointers but have additional metadata and capabilities (like managing memory ownership, counting references, or allowing safe mutation).

> [!NOTE]
> Smart pointers are usually implemented using **structs** that implement two key traits:
> 1. **`Deref`**: Allows smart pointer instances to behave like references so you can use `*` on them.
> 2. **`Drop`**: Allows you to customize the code executed when the smart pointer goes out of scope (memory cleanup!).

## `Box<T>` (Allocating Data on the Heap)

`Box<T>` is the simplest smart pointer. It allows you to store data on the **Heap** rather than the Stack, leaving a small pointer to that heap data on the Stack.

```rust,editable
fn main() {
    // 5 is stored on the Heap, `b` is a Box on the Stack pointing to 5
    let b = Box::new(5);
    println!("b = {b}");
} // 👈 `b` goes out of scope here; both pointer and heap memory are dropped!
```

### When do you use `Box<T>`?

#### Indirection for Recursive Types

Rust must know how much space a type takes up at compile time. Recursive types (where a type has another value of the same type inside itself) have an infinite theoretical size, so the compiler rejects them!

A classic example is a **Cons list** (Lisp-style linked list):

```rust
// ❌ COMPILE ERROR! Recursive type `List` has infinite size!
enum List {
    Cons(i32, List),
    Nil,
}
```

Since a `Box` pointer has a **fixed size** on the stack regardless of what it points to, wrapping the recursion in `Box` solves the problem!

```rust,editable
enum List {
    Cons(i32, Box<List>), // Fixed pointer size!
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

#### Transferring Ownership of Large Data

If you have a massive array or struct on the stack, moving ownership copies all its bytes. Putting it inside a `Box` means moving ownership only copies the small pointer address on the stack!

## The `Deref` Trait (Treating Smart Pointers like References)

Implementing `Deref` lets you customize the behavior of the dereference operator `*`.

```rust,editable
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

// Teach Rust how to dereference MyBox
impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y); // 👈 Rust automatically converts `*y` into `*(y.deref())`!
}
```

### Deref Coercion

Rust automatically converts references to a type implementing `Deref` into a reference to its target type. For example, a `&String` automatically coerces to `&str` because `String` implements `Deref<Target str>`.

## The `Drop` Trait (Running Cleanup Code)

The `Drop` trait lets you specify what happens when a value is about to go out of scope. Rust handles memory cleanup automatically so you don't have to manually call `free()` or `delete()` like in C/C++.

```rust,editable
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    println!("CustomSmartPointers created.");
} 
// Output when main ends:
// CustomSmartPointers created.
// Dropping CustomSmartPointer with data `other stuff`!
// Dropping CustomSmartPointer with data `my stuff`!

```

---

# Chapter 15.3: `Rc<T>` (Reference Counted Smart Pointer)

In standard Rust, a value has a single owner. But what if multiple parts of your program need shared read access to the same heap data?

**`Rc<T>`** (Reference Counting) enables **multiple ownership**. It keeps track of the number of references to a value on the heap. When the count drops to `0`, the memory is cleaned up!

> [!WARNING]
> **Single-Threaded Only:** `Rc<T>` is only for single-threaded scenarios. For multi-threaded ownership, use `Arc<T>` (Atomic Reference Counting, Chapter 16).

```rust,editable
use std::rc::Rc;

enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("Count after creating a = {}", Rc::strong_count(&a)); // 1

    let b = Cons(3, Rc::clone(&a)); // Increments reference count!
    println!("Count after creating b = {}", Rc::strong_count(&a)); // 2

    {
        let c = Cons(4, Rc::clone(&a));
        println!("Count after creating c = {}", Rc::strong_count(&a)); // 3
    } // `c` goes out of scope here

    println!("Count after c goes out of scope = {}", Rc::strong_count(&a)); // 2
}

```

---

# Chapter 15.4: `RefCell<T>` (Interior Mutability Pattern)

Rust's strict borrowing rules enforce:

* Either one mutable reference (`&mut T`), **OR**
* Multiple immutable references (`&T`).

**Interior Mutability** is a design pattern in Rust that allows you to mutate data even when there are immutable references to that data.

`RefCell<T>` enforces borrowing rules **at runtime instead of compile time**:

| Type | Borrow rules checked at... | If rules are broken... |
| --- | --- | --- |
| Standard Reference (`&T`) | **Compile Time** | Code will not compile |
| **`RefCell<T>`** | **Runtime** | Program will panic! (`panic!`) |

```rust,editable
use std::cell::RefCell;

fn main() {
    let data = RefCell::new(5);

    // Borrow mutably at runtime using `.borrow_mut()`
    {
        let mut mut_ref = data.borrow_mut();
        *mut_ref += 10;
    } // `mut_ref` dropped here

    // Borrow immutably at runtime using `.borrow()`
    println!("Value: {:?}", data.borrow()); // 15
}

```

## Combining `Rc<T>` and `RefCell<T>` 🤝

By combining `Rc<T>` and `RefCell<T>`, you can create data that has **multiple owners AND can be mutated by any owner**!

```rust
use std::cell::RefCell;
use std::rc::Rc;

let value = Rc::new(RefCell::new(5));
// Now multiple parts of your code can hold an Rc to `value` 
// and call .borrow_mut() to modify the internal integer!

```

---

# Summary Cheat Sheet

| Smart Pointer | Ownership | Thread Safe? | Mutability | Primary Use Case |
| --- | --- | --- | --- | --- |
| **`Box<T>`** | Single | Yes | Compile-time | Storing data on heap / recursive types |
| **`Rc<T>`** | Multiple | **No** | Immutable | Sharing read-only heap data across scope |
| **`RefCell<T>`** | Single | **No** | Runtime checked | Mutating data behind immutable references |
| **`Rc<RefCell<T>>`** | Multiple | **No** | Runtime checked | Sharing AND mutating shared heap data |
