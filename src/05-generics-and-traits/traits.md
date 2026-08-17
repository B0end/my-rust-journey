# Traits

If Generics allow us to write code for abstract types, **Traits** allow us to define **what those types are capable of doing**.

A trait is similar to an *interface* in languages like Java, TypeScript, or C#, but with Rust's signature compile-time safety and performance.

A **Trait** tells the Rust compiler about functionality a particular type has and can share with other types.

It defines a set of **method signatures** (the function name, parameters, and return type) without necessarily providing the implementation code. Any struct or enum that implements the trait promises to provide those methods!

Suppose we are building a media aggregator app. We have `NewsArticle` structs and `Tweet` structs, and we want both to be able to display a summary.

We define a `Summary` trait using the `trait` keyword:

```rust
pub trait Summary {
    // We only provide the method signature, ending with a semicolon ;
    fn summarize(&self) -> String;
}
```

## Implementing a Trait on a Type

To implement a trait on a struct, use the **`impl Trait for Struct`** syntax:

```rust,editable
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

// Implement Summary for NewsArticle
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

// Implement Summary for Tweet
impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

// Now you can call `.summarize()` on instances of either `NewsArticle` or `Tweet`:

fn main() {
    let tweet = Tweet {
        username: String::from("pepe"),
        content: String::from("Why do programmers prefer dark mode? Because light attracts bugs."),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

Sometimes it’s useful to have a default behavior for a method in a trait, rather than requiring every type to implement it manually.

You provide a method body inside the `trait` block:

```rust,editable
pub trait Summary {
    // Default implementation!
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}

pub struct NewsArticle {
    pub headline: String,
}

// Keep the default behavior by leaving the body empty!
impl Summary for NewsArticle {}

fn main() {
    let article = NewsArticle { headline: String::from("Rust 1.80 Released") };
    println!("{}", article.summarize()); // Prints: "(Read more...)"
}
```

## Traits as Parameters (Trait Bounds)

When a function parameter uses a trait, you're telling Rust: **"I don't care what concrete struct you pass me, as long as it knows how to perform the trait's behavior."**

```rust
// 1. Define the trait with a default implementation
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}

// 2. Define two completely different structs
pub struct NewsArticle {
    pub headline: String,
    pub author: String,
}

pub struct Tweet {
    pub username: String,
    pub content: String,
}

// 3. Implement Summary for NewsArticle (uses default)
impl Summary for NewsArticle {}

// 4. Implement Summary for Tweet (custom implementation)
impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

// --- SYNTAX A: impl Trait (Shorthand) ---
pub fn notify_shorthand(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// --- SYNTAX B: Trait Bound (Explicit Form) ---
pub fn notify_explicit<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

fn main() {
    let article = NewsArticle {
        headline: String::from("Rust 1.80 Released"),
        author: String::from("Jane Doe"),
    };

    let tweet = Tweet {
        username: String::from("rustlang"),
        content: String::from("Rust is awesome!"),
    };

    // Both types work with notify_shorthand!
    notify_shorthand(&article);
    notify_shorthand(&tweet);

    // Both types also work with notify_explicit!
    notify_explicit(&article);
    notify_explicit(&tweet);
}
```

## Combining Multiple Traits (`+` Syntax)

If a function needs a parameter to implement *multiple* traits (e.g., it must be both summarizable AND printable), use `+`:

```rust
use std::fmt::Display;

// 1. Define the Summary trait
pub trait Summary {
    fn summarize(&self) -> String;
}

// 2. Define a struct
pub struct Product {
    pub name: String,
    pub price: f64,
}

// 3. Implement Summary for Product
impl Summary for Product {
    fn summarize(&self) -> String {
        format!("{}: ${:.2}", self.name, self.price)
    }
}

// 4. Implement Display for Product (allows using {} in println!)
impl Display for Product {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "[Product: {}]", self.name)
    }
}

// --- FUNCTION USING COMBINED TRAITS ---
// `item` MUST implement BOTH Summary AND Display
pub fn print_and_summarize(item: &(impl Summary + Display)) {
    // Uses Display (implied by {})
    println!("Printing item: {}", item); 
    
    // Uses Summary
    println!("Summary: {}", item.summarize()); 
}

fn main() {
    let laptop = Product {
        name: String::from("Laptop"),
        price: 999.99,
    };

    print_and_summarize(&laptop);
}
```

## Cleaning Up Trait Bounds with `where` Clauses

When a function has multiple generic parameters with multiple trait bounds, function signatures can get messy:

```rust
// ❌ MESSY & HARD TO READ:
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 { ... }

```

Rust provides a **`where` clause** to move bounds after the function signature:

```rust
// ✅ CLEAN & READABLE:
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // Function body
}
```

## Solving the Mystery from 4.1!

Remember in Section 4.1 when we tried to write a generic `largest` function?

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest { // ❌ COMPILE ERROR!
            largest = item;
        }
    }
    largest
}
```

When you write a generic function like `largest<T>`, Rust doesn't assume **anything** about what type `T` might be. `T` could be an `i32`, a `char`, a `String`, or even a custom struct like `NewsArticle`.

Because not every type in Rust can be compared using the greater-than operator (`>`), Rust blocks `if item > largest` at compile time with an error: **"binary operation `>` cannot be applied to type `&T`"**.

To fix this, we must tell Rust: **"This function only accepts types that know how to compare themselves."** That capability is provided by the **`std::cmp::PartialOrd`** trait.

```rust,editable
use std::cmp::PartialOrd;

// Restricted to types T that implement PartialOrd
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        // Rust allows `>` because PartialOrd guarantees this operation exists!
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    // 1. Works with integers (i32 implements PartialOrd)
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    println!("The largest number is: {}", result);

    // 2. Works with characters (char implements PartialOrd)
    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest(&char_list);
    println!("The largest char is: {}", result);
}
```

## Summary Cheat Sheet

| Syntax | What it does |
| --- | --- |
| **`pub trait TraitName { fn method(&self); }`** | Defines a new trait interface |
| **`impl TraitName for StructName { ... }`** | Implements trait methods for a struct |
| **`pub fn foo(item: &impl TraitName)`** | Accepts any type implementing `TraitName` |
| **`pub fn foo<T: + Trait1 Trait2>(item: &T)`** | Requires type `T` to implement multiple traits |
| **`where T: Trait1`** | Keeps complex trait bounds clean and readable |
