# Asynchronous Programming

In traditional multi-threaded programming, if a thread is waiting for something slow—like reading a file from disk or fetching data from a web API—it sits idle, blocking system resources.

**Asynchronous programming** allows a single thread (or a small pool of threads) to run **thousands of tasks concurrently** by pausing tasks that are waiting for I/O and switching execution to other tasks that are ready to run.

## Synchronous vs. Asynchronous

To understand async code, imagine ordering coffee at a café:

* **Synchronous (Blocking):** The barista takes your order, prepares your coffee while you stand at the counter waiting, gives you your coffee, and only *then* takes the next customer's order.
* **Asynchronous (Non-Blocking):** The barista takes your order, gives you a buzzer, and immediately takes the next customer's order. When your coffee is ready, your buzzer goes off and you pick it up.

```text
Synchronous:   [ Task A Waiting for Network ⏳ ] ---------> [ Resume Task A ]
Asynchronous:  [ Task A Paused ⏸️ ] -> [ Task B Running ⚡ ] -> [ Resume Task A ]
```

## `async` and `.await` Syntax

Adding `async` before a function transforms it into an asynchronous function.

```rust
async fn fetch_data() -> String {
    // ... fetching from network ...
    String::from("Data loaded!")
}
```

> [!IMPORTANT]
> Calling an `async fn` **does not execute its body immediately**! Instead, it returns a value that implements a special trait called a **`Future`** (a value representing work that will finish later).

To actually execute the future and wait for its result, you call **`.await`** on it:

```rust
async fn process() {
    // `.await` pauses execution of `process()` until `fetch_data()` completes,
    // yielding thread control back to the async runtime!
    let data = fetch_data().await; 
    println!("{data}");
}
```

## How Async Works Under the Hood: Futures and Runtimes

Async Rust has a unique architecture compared to languages like JavaScript, Go, or C#.

```
  Your Async Code (`async` / `.await`)
               │
               ▼
   `Future` Trait (Standard Library)  <-- Defines what a Future is
               │
               ▼
   Async Runtime (e.g., Tokio / async-std) <-- Actually executes Futures!
```

### 1. The `Future` Trait (Standard Library)

At the heart of async Rust is the **`Future`** trait (found in `std::future::Future`).

A `Future` is a state machine that can be polled to check its progress:

```rust
pub trait Future {
    type Output;

    // Checks if the future has completed yet
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

pub enum Poll<T> {
    Ready(T), // Computation finished, here is the result!
    Pending,  // Still waiting on I/O. Try again later!
}
```

### 2. Async Runtimes (External Crates)

The Rust standard library **only provides the `Future` trait and `async`/`.await` syntax**. It **does NOT include an async runtime** or event loop!

To run async programs, you need an external runtime crate. The most popular community-standard runtime is **Tokio** (`tokio`).


## A Real Example with Tokio

To write async code in Rust, you add the `tokio` crate to your `Cargo.toml`:

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

Here is a complete async program using Tokio:

```rust,editable
use tokio::time::{sleep, Duration};

async fn fetch_user_data(user_id: u32) -> String {
    println!("Fetching user {user_id}...");
    // Simulate a slow network request without blocking the thread!
    sleep(Duration::from_secs(2)).await; 
    format!("User_{user_id}_Profile")
}

// The `#[tokio::main]` macro sets up the Tokio async runtime event loop!
#[tokio::main]
async fn main() {
    let data = fetch_user_data(42).await;
    println!("Received: {data}");
}
```

## Concurrent Task Execution

The real power of async shines when running multiple operations concurrently!

### A. Spawning Concurrent Tasks (`tokio::spawn`)

You can spawn multiple async tasks to run concurrently on Tokio's background thread pool:

```rust,editable
use tokio::time::{sleep, Duration};

async fn fetch_user_data(user_id: u32) -> String {
    println!("Fetching user {user_id}...");
    // Simulate a slow network request without blocking the thread!
    sleep(Duration::from_secs(2)).await; 
    format!("User_{user_id}_Profile")
}

#[tokio::main]
async fn main() {
    let handle1 = tokio::spawn(async {
        fetch_user_data(1).await
    });

    let handle2 = tokio::spawn(async {
        fetch_user_data(2).await
    });

    // Both requests run concurrently!
    let user1 = handle1.await.unwrap();
    let user2 = handle2.await.unwrap();

    println!("Fetched: {user1} and {user2}");
}
```

### B. Joining Futures (`tokio::join!`)

If you have multiple futures and want to wait for **all** of them to finish concurrently, use the `join!` macro:

```rust,editable
use tokio::time::{sleep, Duration};
use tokio::join;

async fn fetch_user_data(user_id: u32) -> String {
    println!("Fetching user {user_id}...");
    // Simulate a slow network request without blocking the thread!
    sleep(Duration::from_secs(2)).await; 
    format!("User_{user_id}_Profile")
}

#[tokio::main]
async fn main() {
    let f1 = fetch_user_data(1);
    let f2 = fetch_user_data(2);

    // Runs both futures concurrently on the current thread!
    let (user1, user2) = join!(f1, f2);

    println!("Fetched both: {user1}, {user2}");
}
```

## Threads vs. Async Tasks: Which Should You Use?

| Feature | OS Threads (`std::thread`) | Async Tasks (`tokio`) |
| --- | --- | --- |
| **Model** | 1:1 (One OS thread per task) | M:N (Many tasks over few threads) |
| **Resource Cost** | Heavy (~1MB stack per thread) | Extremely Light (~hundreds of bytes) |
| **Scale Limit** | Thousands of threads max | Millions of concurrent tasks |
| **Best For** | **CPU-bound tasks** (image processing, math, AI) | **I/O-bound tasks** (web servers, network sockets, DB queries) |

## Summary Cheat Sheet

| Syntax / Tool | Purpose |
| --- | --- |
| **`async fn foo()`** | Declares an async function that returns a `Future` |
| **`future.await`** | Pauses execution until `future` is resolved |
| **`Future` Trait** | Standard library interface for lazy state machines |
| **`#[tokio::main]`** | Macro that initializes the Tokio runtime and runs `main` |
| **`tokio::spawn`** | Spawns a new concurrent task onto the Tokio background thread pool |
| **`tokio::join!`** | Waits for multiple futures to complete concurrently |
