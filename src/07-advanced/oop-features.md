# Object-Oriented Programming Features

Is Rust an Object-Oriented Programming (OOP) language? **yes and no**.

Rust incorporates core concepts from the OOP paradigm—such as objects, encapsulation, and polymorphism—but it achieves them using **Structs, Enums, Traits, and Trait Objects** rather than classical class-based inheritance.

## Characteristics of Object-Oriented Languages

To see how Rust fits into the OOP paradigm, let's examine the three main pillars of OOP:

### A. Encapsulation

Encapsulation means that the internal implementation details of an object are hidden from code using that object.

In Rust, encapsulation is achieved using the **`pub`** keyword. By default, everything in Rust (struct fields, functions, methods) is private:

```rust,editable
pub struct AveragedCollection {
    list: Vec<i32>,   // Private field!
    average: f64,     // Private field!
}

impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let sum: i32 = self.list.iter().sum();
        self.average = sum as f64 / self.list.len() as f64;
    }
}
```

*External code can call `add()` and `average()`, but it cannot manipulate `list` or `average` directly—guaranteeing that the `average` calculation is never out of sync!*

### B. Inheritance

**Classical Inheritance** allows an object to inherit data and behavior from a parent class.

**Rust does NOT have classical class-based inheritance.**

* You cannot define a struct that inherits fields or methods from another struct.

#### Why didn't Rust include inheritance?

Inheritance has fallen out of favor in modern software design because it often leads to:

* **Rigid hierarchies:** Classes inheriting methods they don't need.
* **Fragile base-class problem:** Changing code in a parent class breaks distant child classes in unexpected ways.

#### Rust's Solution: Composition & Default Trait Methods

Instead of inheritance, Rust encourages **composition** (combining simple structs) and **Traits** to share behavior:

```rust
pub trait Draw {
    fn draw(&self) {
        println!("Default drawing behavior"); // Default implementation!
    }
}
```

### C. Polymorphism

Polymorphism allows code to work with multiple types through a shared interface.

Rust provides **two ways** to achieve polymorphism:

| Polymorphism Type | How Rust implements it | When it happens | Performance |
| --- | --- | --- | --- |
| **Static Polymorphism** | Generics + Trait Bounds (`fn foo<T: Draw>(item: T)`) | **Compile Time** (Monomorphization) | Zero overhead (Fastest) |
| **Dynamic Polymorphism** | **Trait Objects** (`Box<dyn Draw>`) | **Runtime** (Vtable lookup) | Small runtime overhead |

## Using Trait Objects That Allow for Values of Different Types

Suppose you are building a GUI library. You want a `Screen` struct that holds a collection of UI elements (like `Button`, `TextField`, `SelectBox`) and calls a `.draw()` method on each one.

In languages like Java or C#, you would create a parent `Component` class and subclass it.

In Rust, we use **Trait Objects**: **`dyn Trait`**.

#### Step 1: Define the Trait

```rust
pub trait Draw {
    fn draw(&self);
}
```

#### Step 2: Define a Struct using Trait Objects (`dyn Draw`)

Instead of using Generics (which would force the entire vector to contain only *one* concrete type), we store a vector of smart pointers pointing to **`dyn Draw`**:

```rust
pub struct Screen {
    // `Box<dyn Draw>` means "Any heap-allocated type that implements the `Draw` trait"
    pub components: Vec<Box<dyn Draw>>,
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

#### Step 3: Implement the Trait on Different Types

```rust,editable
pub trait Draw {
    fn draw(&self);
}

pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        println!("Drawing a button: {}", self.label);
    }
}

pub struct SelectBox {
    pub width: u32,
    pub options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        println!("Drawing a select box with {} options", self.options.len());
    }
}
```

#### Step 4: Combine Different Types in the Same Vector!

```rust,editable
fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                options: vec![String::from("Yes"), String::from("No")],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

## How Trait Objects Work: Static vs. Dynamic Dispatch

Understanding how Rust handles methods under the hood is key to making good design choices:

```text
Static Dispatch (Generics):
  Compiler creates separate copies of function for each concrete type at compile time.
  `Screen<Button>` ---> Monomorphized Code (Fast, No Runtime Cost)

Dynamic Dispatch (Trait Objects `dyn Trait`):
  Compiler doesn't know concrete types at compile time.
  Pointer ---> Vtable (Virtual Method Table) ---> Actual Method Memory Address
```

* **Static Dispatch (Generics):** The compiler generates code for each specific type at compile time. It enables compiler optimizations like inlining, but forces a collection to hold a single type.
* **Dynamic Dispatch (`dyn Trait`):** Rust uses a **vtable** (virtual method table) at runtime to look up which method to call. This allows heterogeneous collections (mixing `Button` and `SelectBox`), with a tiny runtime lookup penalty.

## Implementing an OOP Design Pattern: The State Pattern

Imagine a blog post workflow:

1. A new post starts as a **Draft**.
2. When the draft is completed, a review is requested (**PendingReview**).
3. Once approved, the post becomes **Published** and its text can be read!

Here is how to structure it using Trait Objects:

```rust,editable
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }

    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }

    pub fn content(&self) -> &str {
        // Asks the current state what text should be visible
        self.state.as_ref().unwrap().content(self)
    }

    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review());
        }
    }

    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve());
        }
    }
}

// State Trait Interface
trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    fn approve(self: Box<Self>) -> Box<dyn State>;
    fn content<'a>(&self, _post: &'a Post) -> &'a str {
        "" // Default: draft and pending review posts return an empty string!
    }
}

// 1. Draft State
struct Draft {}
impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self // Cannot approve a draft directly
    }
}

// 2. Pending Review State
struct PendingReview {}
impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
    fn approve(self: Box<Self>) -> Box<dyn State> {
        Box::new(Published {})
    }
}

// 3. Published State
struct Published {}
impl State for Published {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content // Published posts expose their content!
    }
}

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content()); // Draft post returns empty text!

    post.request_review();
    assert_eq!("", post.content()); // Pending review post returns empty text!

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content()); // Published!
}
```

## Idiomatic Rust Alternative: Encoding States as Types

While the traditional OOP State Pattern above works in Rust, it relies on runtime polymorphism (`Box<dyn State>`) and `Option::take()`.

An **idiomatic Rust approach** uses the type system to enforce states at **compile time**:

```rust,editable
pub struct DraftPost {
    content: String,
}

pub struct PendingReviewPost {
    content: String,
}

pub struct Post {
    content: String,
}

impl DraftPost {
    pub fn new() -> DraftPost {
        DraftPost { content: String::new() }
    }

    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }

    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost { content: self.content }
    }
}

impl PendingReviewPost {
    pub fn approve(self) -> Post {
        Post { content: self.content }
    }
}

impl Post {
    pub fn content(&self) -> &str {
        &self.content
    }
}

fn main() {
    let mut post = DraftPost::new();
    post.add_text("I ate a salad for lunch today");

    let pending_post = post.request_review();
    let published_post = pending_post.approve();

    println!("Content: {}", published_post.content());
    
    // ❌ COMPILE ERROR if you try to call `.content()` on a DraftPost or PendingReviewPost!
    // The compiler prevents calling invalid methods for a given state!
}
```

## Summary Cheat Sheet

| OOP Concept | Classical OOP | Rust Approach |
| --- | --- | --- |
| **Objects & Data** | Classes with attributes | `struct` / `enum` |
| **Methods** | Class methods | `impl StructName { fn method(&self) }` |
| **Encapsulation** | `private`, `protected`, `public` | Module system + `pub` keyword |
| **Inheritance** | Class hierarchies (`class Child extends Parent`) | **Not supported.** Use composition & Trait default methods |
| **Heterogeneous Collections** | Subtyping polymorphism | Trait Objects (`Vec<Box<dyn Trait>>`) |
| **Dispatch Model** | Virtual tables by default | Generics (Static) by default, `dyn Trait` (Dynamic) on demand |

---
