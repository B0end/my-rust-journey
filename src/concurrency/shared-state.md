# Shared-State

Message passing is great, but sometimes multiple threads need to access and modify the same memory location simultaneously.

To do this safely, Rust uses two smart pointers together:

1. **`Mutex<T>` (Mutual Exclusion):** Guarantees that only one thread can access data at a time.
2. **`Arc<T>` (Atomic Reference Counting):** Allows multiple threads to safely share ownership of the `Mutex`.

A **Mutex** acts like a lock. To access the data inside a `Mutex<T>`, a thread must first ask to **lock** it (`.lock()`). Once locked, no other thread can access the data until the lock is released when the returned `MutexGuard` goes out of scope!

```rust,editable
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        // `.lock()` returns a MutexGuard smart pointer
        let mut num = m.lock().unwrap();
        *num = 6;
    } // Lock is automatically released here when `num` goes out of scope!

    println!("m = {m:?}");
}
```

`Rc<T>` enables multiple ownership, but **it is not thread-safe**.

To share ownership across threads, we use **`Arc<T>`** (**A**tomic **R**eference **C**ounted smart pointer):

```rust,editable
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // Create a thread-safe atomic reference counter wrapping a Mutex
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter_clone = Arc::clone(&counter);
        
        let handle = thread::spawn(move || {
            let mut num = counter_clone.lock().unwrap();
            *num += 1; // Thread-safe mutation!
        });
        
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap()); // Prints 10!
}
```

## Extensible Concurrency with `Send` and `Sync` Traits

Unlike many languages, thread safety features in Rust aren't hardcoded into the language core—they are defined by two standard library marker traits:

1. `Send` Trait

Indicates that **ownership of a type can be transferred across thread boundaries**.

* Most standard library types are `Send` (like `i32`, `String`, `Vec<T>`).
* Types like `Rc<T>` are **NOT** `Send` because thread-unsafe reference counting across threads causes data corruption.

2. `Sync` Trait

Indicates that **it is safe for multiple threads to access references (`&T`) of the type**.

* A type `T` is `Sync` if `&T` is `Send`.
* Primitive types are `Sync`. `Mutex<T>` and `Arc<T>` are `Sync`.
* `RefCell<T>` and `Rc<T>` are **NOT** `Sync`.

> [!NOTE]
> You almost never need to implement `Send` or `Sync` manually! Any struct made entirely of types that implement `Send` and `Sync` automatically inherits them.

## Summary Cheat Sheet

| Primitive | What it does | Key Characteristic |
| --- | --- | --- |
| **`thread::spawn`** | Creates a new OS thread | Runs code concurrently |
| **`move \|\| ...`** | Forces closure to take ownership | Required when moving variables into threads |
| **`mpsc::channel()`** | Creates a message passing channel | Multiple senders (`tx`), single receiver (`rx`) |
| **`Mutex<T>`** | Provides mutual exclusion | Enforces runtime locks for safe thread mutation |
| **`Arc<T>`** | Atomic reference counting | Thread-safe version of `Rc<T>` for multi-ownership |
| **`Send` / `Sync`** | Marker traits | Ensured by compiler for thread safety guarantees |
