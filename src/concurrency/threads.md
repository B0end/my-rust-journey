# Threads

By default, when you run a program, your operating system executes code sequentially in a single **thread of execution**. Think of a thread as a single sequence of instructions executed line by line.

If your computer has multiple CPU cores (most modern devices have 4, 8, or more), running a program in a single thread leaves those extra cores idle.

Why Use Threads?

* **Simultaneous Execution (Concurrency / Parallelism):** Splitting your program into multiple threads allows different tasks to run in parallel on separate CPU cores (e.g., loading game textures on one thread while calculating physics on another).
* **Non-Blocking Work:** If one thread has to wait for slow operations like network requests or file downloads, other threads can keep running without freezing the entire application.

To create a new thread in Rust, call **`thread::spawn`** and pass it a closure containing the code you want the thread to run:

```rust,editable
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..5 {
            println!("hi number {i} from the spawned thread!");
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..3 {
        println!("hi number {i} from the main thread!");
        thread::sleep(Duration::from_millis(1));
    }
}

```

> [!WARNING]
> When the `main` thread finishes, **all spawned threads are stopped immediately**, whether they have finished their execution or not!

To ensure a spawned thread completes before `main` exits, save its return value—a **`JoinHandle`**—and call **`.join()`** on it:

```rust,editable
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        println!("Hello from spawned thread!");
    });

    // .join() blocks the main thread until the spawned thread completes!
    handle.join().unwrap();

    println!("Hello from main thread!");
}
```

## Using `move` Closures with Threads

If a spawned thread needs data from the main thread's scope, Rust forces you to transfer ownership using the **`move`** keyword. This prevents the spawned thread from referencing data that might get dropped by the main thread while the spawned thread is still running!

Without the `move` keyword, the closure only tries to **borrow** `v` from the main thread. If Rust allowed that, the main thread could modify `v` (creating a **data race**) or drop `v` entirely before the thread finishes running (causing a **dangling pointer** or **use-after-free** memory bug).

Because a spawned thread runs concurrently and its lifetime is independent of the main thread's local scope, Rust's borrow checker strictly enforces two things:

1. Dropping / Outliving the Variable

Without `move`, the main thread could exit or clear `v` from memory while the spawned thread is still executing:

```rust
let v = vec![1, 2, 3];

let handle = thread::spawn(|| {
    // ❌ Rust won't compile this!
    // What if the main thread drops `v` right here before this line executes?
    println!("{v:?}"); 
});

drop(v); // Main thread frees the vector's memory
handle.join().unwrap();
```

2. Data Races and Mutability

Even if you don't drop `v`, trying to modify it in the main thread while another thread reads it creates a classic data race:

```rust
let mut v = vec![1, 2, 3];

let handle = thread::spawn(|| {
    println!("{v:?}"); // Thread tries to read `v`
});

v.push(4); // ❌ Main thread reallocates memory for `v` at the same time!
handle.join().unwrap();
```

When you write `move ||`, the main thread transfers **complete ownership** of `v` to the spawned thread's closure. Once moved:

* The main thread can no longer read, modify, or drop `v`.
* The compiler guarantees that `v` lives as long as the spawned thread needs it.

```rust
let mut v = vec![1, 2, 3];

let handle = thread::spawn(move || {
    println!("{v:?}"); // `v` belongs entirely to this thread now
});

// v.push(4); ❌ Compile error! `v` no longer exists in `main`'s scope.

handle.join().unwrap();
```

## Message Passing with Channels (`mpsc`)

A popular approach to safe concurrency is **message passing**, where threads communicate by sending messages to each other rather than sharing memory directly.

Rust's standard library provides an **`mpsc`** channel implementation, which stands for **Multiple Producer, Single Consumer**:

* **Multiple Producer:** Multiple threads can create clones of the transmitter to *send* messages into the same channel.
* **Single Consumer:** Only one thread can hold the receiver and *receive* messages from the channel.

```rust,editable
use std::sync::mpsc;
use std::thread;

fn main() {
    // 1. CREATING THE CHANNEL
    // `mpsc::channel()` returns a tuple containing a Transmitter (tx) and a Receiver (rx).
    // The compiler automatically infers the data type passed through the channel (here, `String`).
    let (tx, rx) = mpsc::channel();

    // 2. OWNERSHIP & THREAD SPAWNING
    // We use the `move` keyword to transfer ownership of `tx` into the spawned thread's closure.
    thread::spawn(move || {
        let val = String::from("hi");
        
        // 3. SENDING DATA & RESULT HANDLING
        // `tx.send(val)` returns a `Result<(), SendError<T>>`.
        // - `Ok(())`: Message was successfully queued into the channel.
        // - `Err(SendError)`: Occurs if the receiver (`rx`) was dropped by the main thread.
        tx.send(val).unwrap(); 
        
        // 4. PREVENTING DATA RACES VIA OWNERSHIP TRANSFER
        // ❌ COMPILE ERROR! `val` was MOVED into `tx.send()`.
        // Rust's borrow checker prevents you from reading `val` after sending it, 
        // ensuring no other thread can mutate or read memory simultaneously.
        // println!("val is {val}"); 
    });

    // 5. RECEIVING DATA: `recv()` vs `try_recv()`
    // `rx.recv()` BLOCKS the main thread (puts it to sleep) until a message arrives.
    // It returns a `Result<T, RecvError>`:
    // - `Ok(received)`: Successfully retrieved the message.
    // - `Err(RecvError)`: Occurs when ALL transmitters (`tx`) have been dropped,
    //   signaling no further messages will ever arrive.
    let received = rx.recv().unwrap();
    println!("Got: {received}");
}
```

Both `send()` and `recv()` return Rust's standard `Result` enum (`Result<T, E>`):

* **`tx.send(val)`** ➡️ `Result<(), SendError<T>>`
* **`rx.recv()`** ➡️ `Result<T, RecvError>`

Using `.unwrap()` forces the program to extract the inner value on success, or panic (crash) on error. While common in short examples, real-world Rust code uses pattern matching (`match` or `if let`) or the `?` operator to gracefully handle disconnected channels.

### Blocking vs. Non-Blocking: `recv()` vs. `try_recv()`

The receiver (`rx`) gives you two ways to fetch messages:

| Method | Behavior | Return Type | Best Used When... |
| --- | --- | --- | --- |
| **`recv()`** | **Blocking:** Pauses the thread until a message is sent or the channel closes. | `Result<T, RecvError>` | The current thread has no other tasks and must wait for input. |
| **`try_recv()`** | **Non-blocking:** Returns immediately without waiting. | `Result<T, TryRecvError>` | A loop needs to check for messages periodically while continuing other work. |

When using `try_recv()`, the error variant is an enum (`TryRecvError`) with two cases:

* **`TryRecvError::Empty`**: No message available *yet*, but the channel is still open.
* **`TryRecvError::Disconnected`**: All transmitters have been dropped; no future messages are possible.

## Sending Multiple Values

Instead of sending a single message and exiting, a thread can stream a series of values over time.

Treating the receiver (`rx`) as an **iterator** in a `for` loop automatically calls `recv()` behind the scenes on each iteration. The loop will block to wait for incoming messages and will automatically terminate once all transmitters are dropped.

```rust,editable
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            // Pause briefly to simulate work or time passing
            thread::sleep(Duration::from_secs(1));
        }
        // `tx` goes out of scope here and is dropped automatically!
    });

    // Treating `rx` as an iterator consumes incoming messages one by one.
    // The loop automatically stops when the sender `tx` is dropped.
    for received in rx {
        println!("Got: {received}");
    }
}
```

## Cloning the Transmitter for Multiple Producers

Because `mpsc` allows multiple senders, you can clone `tx` to send messages from multiple threads into a single receiver:

```rust,editable
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone(); // Clone sender for Thread 1

    thread::spawn(move || {
        tx1.send("hello from thread 1").unwrap();
    });

    thread::spawn(move || {
        tx.send("hello from thread 2").unwrap();
    });

    // Treat the receiver as an iterator to process incoming messages
    for received in rx {
        println!("{received}");
    }
}
```