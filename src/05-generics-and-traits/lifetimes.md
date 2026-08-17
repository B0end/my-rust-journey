# Validating References with Lifetimes

A **dangling reference** happens when a program holds a reference to memory that gets dropped (freed) while the reference is still alive.

Consider this invalid code:

```rust,editable
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // --+-- 'b |
        r = &x;           //   |      |
    }                     // --+      |  <-- `x` dies here!
                          //          |
    println!("r: {r}");   //          |  <-- `r` tries to use dead memory!
}                         // ---------+
```

The Rust compiler tracks the scope of every variable using **Lifetimes** (labeled `'a` and `'b` in the comments above):

* `r` has lifetime `'a` (lives through the whole `main` function).
* `x` has lifetime `'b` (only lives inside the inner block `{ }`).

Because lifetime `'b` is **shorter** than lifetime `'a`, the compiler rejects this code at compile time with: `error: x does not live long enough`.


## When Do We Need Lifetime Annotations?

Most of the time, Rust automatically infers lifetimes (via *Lifetime Elision Rules*). However, when a function accepts **two or more references** and returns a reference, Rust needs your help to know which input reference the output reference is connected to!

Look at this function that finds the longer of two string slices:

```rust,editable
// ❌ COMPILE ERROR! Rust doesn't know which lifetime the returned reference belongs to!
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {}
```

Why does this fail?
If `x` and `y` come from different scopes, Rust doesn't know at compile time whether the returned slice points to `x` or `y`. Therefore, it can't verify if the returned reference will remain valid!

## Generic Lifetime Annotation Syntax

Lifetime annotations **do not change how long a variable lives**. Instead, they describe relationships between the lifetimes of multiple references.

* Lifetime names start with an apostrophe `'`.
* By convention, short lowercase names are used (usually `'a`, `'b`, `'c`).
* Place them inside angle brackets `<'a>` after the function name, just like generic types!

Here is the fixed function:

```rust,editable
// Read as: "The returned reference will live as long as the SHORTEST of 'x' or 'y'"
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {}
```

`'a` tells the Borrow Checker:

> *"Take two references `x` and `y`. Find whichever scope is **shorter**, call that lifetime `'a'`, and promise that the returned reference will remain valid for at least `'a'`."*

Let's test it in `main()`:

```rust,editable
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {result}"); // ✅ VALID! Both string1 and string2 are alive here.
    }
}
```

If you try to print `result` **outside** the inner scope where `string2` died, the compiler will catch it and block the build!

## Thinking in Terms of Lifetimes

When annotating lifetimes, ask yourself: **How does the output reference relate to the input parameters?**

* If a function returns a reference, its lifetime **MUST** match one of the input parameters.
* If it returns a reference to something created *inside* the function, it's a dangling reference (and won't compile!):

```rust
// ❌ FAILS! `result` dies when the function ends, returning a pointer to garbage!
fn invalid_fn<'a>() -> &'a String {
    let result = String::from("really long string");
    &result 
}
```

*(Fix: Return the owned `String` value directly instead of a reference `&String`!)*

## Lifetimes in Struct Definitions

If a struct holds an **owned** type (like `String` or `i32`), you don't need lifetimes. But if a struct holds a **reference** (`&str`), you must annotate it so the struct cannot outlive the data it references!

```rust,editable
// Read as: "An instance of ImportantExcerpt cannot outlive the reference in `part`"
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");

    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

## Lifetime Elision Rules

Why haven't you had to write `'a` everywhere in previous chapters?

The Rust compiler has **three deterministic rules** hardcoded into it. If your code fits these patterns, Rust inserts the lifetimes automatically (*Lifetime Elision*):

1. **Rule 1:** Each parameter that is a reference gets its own lifetime parameter (`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`).
2. **Rule 2:** If there is exactly one input lifetime parameter, that lifetime is assigned to *all* output references (`fn foo<'a>(x: &'a i32) -> &'a i32`).
3. **Rule 3:** If there are multiple input lifetime parameters, but one of them is `&self` or `&mut self` (a method), the lifetime of `self` is assigned to all output references.

## The `'static` Lifetime

There is one special lifetime reserved by Rust: **`'static`**.

A `'static` reference can live for the **entire duration of the program**.

All **string literals** implicitly have a `'static` lifetime because their raw text is baked directly into the compiled program's binary code:

```rust
let s: &'static str = "I have a static lifetime!";
```

## Summary Cheat Sheet

| Concept | Syntax / Code | Meaning |
| --- | --- | --- |
| **Reference with Lifetime** | `&'a i32` | A reference to `i32` with lifetime `'a` |
| **Mutable Reference + Lifetime** | `&'a mut i32` | A mutable reference to `i32` with lifetime `'a` |
| **Generic Lifetime Function** | `fn foo<'a>(x: &'a str) -> &'a str` | Connects input lifetime to output lifetime |
| **Struct with Reference** | `struct Foo<'a> { bar: &'a str }` | Struct cannot outlive the reference inside `bar` |
| **Static Lifetime** | `&'static str` | Reference lives for the entire program duration |
