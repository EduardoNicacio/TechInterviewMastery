# Rust – Interview Questions & Answers

**Question**: What are Rust's ownership rules, and why are they fundamental to memory safety?

**Answer**: Every value in Rust has exactly one owner (a variable holding it). When the owner goes out of scope, the value is dropped — no garbage collector needed. Ownership can be moved to another owner, but at any time there is precisely one binding responsible for the value's lifetime.

```rust
let s1 = String::from("hello");
let s2 = s1;          // ownership moved to s2
// println!("{s1}");  // compile error: s1 no longer valid
```

---

**Question**: How do references (`&T`) and mutable references (`&mut T`) differ from owned values?

**Answer**: References borrow a value without taking ownership, allowing access without transferring ownership. `&T` grants shared read-only access, while `&mut T` grants exclusive read-write access. References must always point to a valid value for the duration of the borrow.

```rust
fn read_len(s: &String) -> usize { s.len() }
fn append(s: &mut String) { s.push_str("!"); }
```

---

**Question**: What are Rust's borrowing rules, and what do they guarantee at compile time?

**Answer**: At any given scope, you may have either one mutable reference or any number of immutable references, but never both simultaneously. The compiler enforces this to prevent data races and aliased mutation entirely at compile time.

```rust
let mut s = String::from("hi");
let r1 = &s;
let r2 = &s;
// let r3 = &mut s; // error: cannot borrow as mutable because also borrowed as immutable
println!("{r1} {r2}");
```

---

**Question**: What are slices (`&str`, `&[T]`) and how do they relate to borrowing?

**Answer**: A slice is a dynamically-sized view into a contiguous sequence — it borrows a portion of a collection without owning it. `&str` is a string slice referencing part of a `String`, and `&[T]` is an array or `Vec<T>` slice. Slices guarantee the data remains valid through their lifetime.

```rust
let s = String::from("hello world");
let hello = &s[0..5];        // &str slice
let bytes = &[1, 2, 3, 4];
let mid = &bytes[1..3];      // &[i32] slice
```

---

**Question**: What are lifetime annotations (`'a`) and when must they be explicit?

**Answer**: Lifetime annotations connect the lifetimes of references to ensure they outlive each other. They appear as generic parameters like `'a` on functions, structs, or impls. The compiler infers them in most cases; explicit annotations are needed when a function returns a reference whose lifetime cannot be deduced.

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

---

**Question**: What are Rust's lifetime elision rules?

**Answer**: The compiler automatically infers lifetimes on function signatures when three patterns hold: each input reference gets its own lifetime, if there is exactly one input lifetime it is assigned to all output references, and if `&self` or `&mut self` is present its lifetime is assigned to all output references. These rules cover the vast majority of practical cases.

---

**Question**: What are non-lexical lifetimes (NLL) and how did they improve Rust?

**Answer**: NLL allows borrows to be released as soon as they are last used, rather than lasting until the end of the enclosing scope. This was introduced in Rust 2018 and eliminates many frustrating borrow-checker errors where a borrow was held longer than logically necessary, enabling more idiomatic code.

```rust
let mut s = String::from("hi");
let r = &s;
println!("{r}");
// NLL: r is dropped here, so mutable borrow below is fine
let r2 = &mut s;
r2.push_str("!");
```

---

**Question**: How does interior mutability work with `Cell<T>` and `RefCell<T>`?

**Answer**: `Cell<T>` and `RefCell<T>` provide interior mutability — mutation through an immutable reference (`&T`). `Cell<T>` uses value-level moves (`get`/`set`) and works for any `T`; only `get()` requires `T: Copy`, while `set()`, `replace()`, and `take()` work with any type. `RefCell<T>` enforces borrowing rules at runtime via `borrow()`/`borrow_mut()`, returning `Ref`/`RefMut` guards, and panics if the rules are violated.

```rust
use std::cell::RefCell;
let data = RefCell::new(42);
*data.borrow_mut() += 1;
println!("{}", data.borrow());
```

---

**Question**: What are `Rc<T>` and `Arc<T>`, and when would you use each?

**Answer**: `Rc<T>` provides single-threaded reference-counted ownership (clone increments a counter, drop decrements it, the value is freed when the count reaches zero). `Arc<T>` is the thread-safe counterpart using atomic operations. Use `Rc` for shared ownership within one thread, `Arc` for shared ownership across threads, typically with `Mutex` or `RefCell` for mutation.

```rust
use std::rc::Rc;
let a = Rc::new(5);
let b = Rc::clone(&a);   // reference count = 2
```

---

**Question**: What is `Cow` (clone-on-write) and how does it optimize performance?

**Answer**: `Cow<'a, B>` is an enum that can hold either a borrowed reference (`Borrowed`) or an owned value (`Owned`). It avoids cloning until mutation is actually requested — at which point it clones the borrowed variant into an owned one. It is ideal for functions that mostly return static data but need to handle dynamic cases without always allocating.

```rust
use std::borrow::Cow;
fn normalize(input: &str) -> Cow<str> {
    if input.contains(' ') {
        input.replace(' ', "_").into()  // Owned
    } else {
        Cow::Borrowed(input)            // No allocation
    }
}
```

---

**Question**: How do you define and implement traits in Rust, and what are blanket impls?

**Answer**: Traits define shared behavior via method signatures. They are implemented on types using `impl Trait for Type`. Blanket impls implement a trait for any type satisfying a bound — e.g., `impl<T: Display> ToString for T`. They enable powerful extensibility but require care to avoid coherence conflicts.

```rust
trait Greet {
    fn greet(&self) -> String;
}
impl Greet for &str {
    fn greet(&self) -> String { format!("Hello, {self}!") }
}
```

---

**Question**: How do generics and trait bounds work in Rust?

**Answer**: Generics allow functions and types to operate over multiple concrete types. Trait bounds constrain which types are accepted, enabling monomorphization — the compiler generates separate optimized code for each concrete type. Bounds use `T: Trait` syntax or the `where` clause for complex constraints.

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    list.iter().max_by(|a, b| a.partial_cmp(b).unwrap()).unwrap()
}
```

---

**Question**: What is the difference between `dyn Trait` and `impl Trait`?

**Answer**: `impl Trait` is a compile-time construct that uses static dispatch — the concrete type is known at compile time, enabling inlining and no vtable overhead. `dyn Trait` is a dynamically-sized trait object that uses dynamic dispatch through a vtable, incurring a pointer indirection but allowing heterogeneous collections. Use `impl Trait` in argument/return positions for performance; use `dyn Trait` when you need type erasure.

```rust
fn dynamic(animals: &[&dyn Animal]) { /* vtable dispatch */ }
fn static_single(a: &impl Animal) {   /* monomorphized */ }
```

---

**Question**: When should you use associated types versus generic parameters on a trait?

**Answer**: Use associated types when a trait has exactly one natural output type (e.g., `Iterator::Item`) — the implementor fixes the type, and callers don't parameterize it. Use generic parameters when a trait can sensibly be implemented multiple times for the same type with different type arguments (e.g., `From<T>`). Associated types simplify usage since the bound is single-valued.

```rust
trait Iterator {
    type Item;           // associated type — one per impl
    fn next(&mut self) -> Option<Self::Item>;
}
trait From<T> {          // generic — can impl From<A> and From<B> for same type
    fn from(t: T) -> Self;
}
```

---

**Question**: What are the `Sized` and `?Sized` markers, and what is `PhantomData` used for?

**Answer**: `Sized` marks types with a known compile-time size; all type parameters are `Sized` by default. `?Sized` relaxes this constraint, allowing dynamically-sized types like `[u8]` or `dyn Trait`. `PhantomData<T>` is a zero-sized marker used in structs to simulate ownership of a type for drop-check, variance, and type-state programming without runtime overhead.

```rust
use std::marker::PhantomData;
struct RawPtr<T: ?Sized> {
    ptr: *const T,
    _marker: PhantomData<T>,  // tells drop check we own a T
}
```

---

**Question**: How do `From`/`Into` and `TryFrom`/`TryInto` conversion traits work?

**Answer**: `From<T>` defines infallible conversion from `T` to `Self`, and `Into<T>` is the reciprocal (automatically provided via a blanket impl). `TryFrom`/`TryInto` handle fallible conversions, returning `Result`. Implementing `From` is preferred — it provides `Into` for free and enables idiomatic `.into()` usage.

```rust
struct MyType(String);

impl From<&str> for MyType {
    fn from(s: &str) -> Self { MyType(s.to_string()) }
}
let x: MyType = "hello".into();
```

---

**Question**: What are `Send` and `Sync`, and how does Rust enforce thread safety with them?

**Answer**: `Send` indicates a type's ownership can be transferred across threads safely. `Sync` indicates a reference `&T` can be shared across threads safely. These are auto-traits — the compiler implements them automatically for types composed of `Send`/`Sync` parts. Most types are `Send` + `Sync`, but `Rc<T>`, `RefCell<T>`, and raw pointers are not, preventing data races at compile time.

---

**Question**: How do `Mutex<T>` and `RwLock<T>` provide shared mutation across threads?

**Answer**: `Mutex<T>` serializes access via `.lock()` which returns a `MutexGuard` that derefs to `&mut T`. `RwLock<T>` allows concurrent reads but exclusive writes via `.read()`/`write()`. Both are typically wrapped in `Arc` for shared ownership. The guard's `Drop` implementation automatically releases the lock, making deadlocks unlikely with single-lock patterns.

```rust
use std::sync::{Arc, Mutex};
let counter = Arc::new(Mutex::new(0));
* counter.lock().unwrap() += 1;
```

---

**Question**: How do channels in `std::sync::mpsc` work for message passing?

**Answer**: `std::sync::mpsc` provides a multi-producer, single-consumer channel. The sender (`Sender`) can be cloned for multiple producers, and the receiver (`Receiver`) receives messages via `recv()` (blocking) or `try_recv()` (non-blocking). Each message is moved across the channel, avoiding shared-state contention — a fundamental concurrency paradigm.

```rust
use std::sync::mpsc;
let (tx, rx) = mpsc::channel();
tx.send(42).unwrap();
assert_eq!(rx.recv().unwrap(), 42);
```

---

**Question**: What does the Crossbeam library provide beyond `std` concurrency primitives?

**Answer**: Crossbeam offers scoped threads (threads that borrow local variables without `'static` bounds), work-stealing deques for task schedulers, epoch-based reclamation for lock-free data structures, and multi-producer multi-consumer (MPMC) channels. Scoped threads are particularly useful for parallel algorithms that reference stack data.

```rust
// Prefer std::thread::scope (Rust 1.63+) for new code:
std::thread::scope(|s| {
    s.spawn(|| println!("child"));
    s.spawn(|| println!("another child"));
});
// crossbeam::scope still works but std::thread::scope is now preferred
```

---

**Question**: How does `async`/`await` work in Rust, and what is the `Future` trait?

**Answer**: `Future` is a trait with one method — `poll(self: Pin<&mut Self>, cx: &mut Context) -> Poll<T>` — representing an asynchronous computation that may not be ready yet. The `async fn` keyword desugars a function into a state machine implementing `Future`. `.await` yields control back to the executor, allowing other tasks to run while waiting.

```rust
use std::future::Future;
async fn fetch_data() -> String {
    "data".to_string()
}
```

---

**Question**: What are `Pin` and `Unpin`, and how do they relate to async Rust and Tokio?

**Answer**: `Pin<P>` is a wrapper that prevents the pointee from being moved out of its memory location. `Unpin` is an auto-trait — most types implement it, allowing moves freely. Futures generated by `async fn` are not `Unpin` because they may contain self-referential structs across `.await` points. Tokio's `spawn` requires `Send + 'static` futures, and `Pin` ensures the future's internal state is stable while polled.

---

**Question**: What is the `?` operator and how does it work with `Result<T, E>` and `Option<T>`?

**Answer**: The `?` operator unwraps a `Result` or `Option` — for `Ok`/`Some` it yields the inner value; for `Err`/`None` it returns early from the enclosing function, converting the error via `From`. It eliminates nested match boilerplate and is the idiomatic way to handle fallible operations in Rust.

```rust
use std::fs;
use std::io::{self, Read};

fn read_file(path: &str) -> Result<String, io::Error> {
    let mut f = fs::File::open(path)?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

---

**Question**: How do you define custom error types and what do `thiserror` and `anyhow` provide?

**Answer**: Custom error types implement `std::error::Error` (which requires `Display` and `Debug`). `thiserror` is a derive macro that auto-generates `Display` and `Error` impls with `#[error("...")]` annotations, ideal for library crates. `anyhow::Error` is an opaque error type with `.context()` for attaching messages, preferred in application code for convenience.

```rust
use thiserror::Error;
#[derive(Error, Debug)]
pub enum ParseError {
    #[error("invalid UTF-8 at offset {offset}")]
    InvalidUtf8 { offset: usize },
}
```

---

**Question**: How do you decide between `panic!` and `Result` for error handling?

**Answer**: `panic!` is for unrecoverable errors — bugs, logic violations, or states where continuing would cause undefined behavior (e.g., out-of-bounds indexing by a known-valid index). `Result` is for recoverable errors — I/O failures, parse errors, user input — where the caller should decide how to handle the failure. Panics unwind the stack (or abort), so they should not be used for expected failure modes.

---

**Question**: When is it acceptable to use `unwrap()` or `expect()`?

**Answer**: Use `unwrap()` or `expect()` when you are certain the computation cannot fail due to invariants the compiler cannot prove — e.g., a known-valid static value, a test, or an infallible operation like locking a non-poisoned mutex after setup. `expect()` is preferred because it documents the invariant. In production library code, return `Result` instead.

```rust
let val: u32 = "42".parse().expect("hardcoded integer literal");
```

---

**Question**: What are zero-cost abstractions in Rust, and how does trait monomorphization achieve them?

**Answer**: Zero-cost abstractions mean that higher-level code compiles to the same machine code as hand-written lower-level code — you pay only for what you use. Trait monomorphization generates separate concrete implementations for each type parameter at compile time, allowing full inlining and specialization with no vtable or dynamic dispatch overhead.

---

**Question**: How do you call C functions from Rust using FFI (`extern "C"`, `#[no_mangle]`)?

**Answer**: FFI declarations use `extern "C"` blocks to declare foreign function signatures, wrapped in `unsafe` because the caller must uphold C invariants (valid pointers, correct types). Rust functions exposed to C must be `#[no_mangle]` (prevent name mangling) and `extern "C"` for C-ABI compatibility.

```rust
use std::ffi::{c_char, CString};

extern "C" {
    fn strlen(s: *const c_char) -> usize;
}
unsafe { strlen(CString::new("hi").unwrap().as_ptr()); }
```

---

**Question**: What are Rust's unsafe superpowers, and when should you use them?

**Answer**: Unsafe enables five operations: dereference raw pointers, call unsafe functions, access/modify mutable statics, implement unsafe traits, and access fields of unions. Use unsafe only when you can prove invariants the compiler cannot (e.g., at FFI boundaries, implementing lock-free data structures, or for performance-critical SIMD). Every unsafe block should have a safety comment justifying its correctness.

```rust
let x = 42;
let r = &x as *const i32;
unsafe { println!("{}", *r); } // safe: r points to a valid, aligned i32
```

---

**Question**: What are raw pointers (`*const T`, `*mut T`) and how do they differ from references?

**Answer**: Raw pointers are unrestricted pointers that can be null, dangling, or aliased mutably — they lack the safety guarantees of references. `*const T` and `*mut T` correspond to `&T` and `&mut T` but can be created in safe code. Dereferencing them requires `unsafe`, and they are `!Send + !Sync` by default, placing correctness entirely on the programmer.

```rust
let mut x = 10;
let p: *mut i32 = &mut x;
unsafe { *p = 20; }
assert_eq!(x, 20);
```

---

**Question**: What are `repr(C)` and `repr(Rust)` memory layouts, and why does `repr(C)` matter for FFI?

**Answer**: `repr(Rust)` (the default) allows the compiler to reorder fields, add padding, and optimize layout freely. `repr(C)` forces a C-compatible layout with fields in declaration order, which is required when passing structs across FFI boundaries. It also enables transmutation between structs with the same fields (if no enum discriminants are involved).

```rust
#[repr(C)]
struct Point { x: f64, y: f64 }
```

---

**Question**: What does `no_std` mean, and when would you use it?

**Answer**: `#![no_std]` removes the standard library dependency, linking only the `core` crate by default. The `alloc` crate is opt-in via `extern crate alloc;`. It is used in embedded systems, kernels, bootloaders, and WebAssembly targets where the OS services (heap, threads, I/O) are unavailable. You must provide your own allocator and often implement panic handling and platform-specific abstractions.

```rust
#![no_std]
extern crate alloc;
use alloc::vec::Vec;
```

---

**Question**: How do Cargo workspaces and features help organize Rust projects?

**Answer**: A workspace is a set of crates sharing a single `Cargo.lock` and output directory, enabling coordinated builds across library/application crates. Feature flags (`[features]` in `Cargo.toml`) enable conditional compilation — crates can expose optional functionality that consumers toggle via `features = ["serde"]`, allowing dependency trees to remain lean.

```toml
[workspace]
members = ["core", "cli", "web"]
[features]
default = ["std"]
std = ["core/std"]
```

---

**Question**: How does `serde` provide serialization/deserialization in Rust?

**Answer**: `serde` is a framework with a derive macro (`#[derive(Serialize, Deserialize)]`) that auto-generates implementations for data formats like JSON, YAML, and Bincode. The derive generates an impl of `Serialize` (visitor pattern) and `Deserialize` (extraction via `Deserializer`). Custom serialization can be achieved by manually implementing the traits.

```rust
use serde::{Serialize, Deserialize};
#[derive(Serialize, Deserialize)]
struct Config { host: String, port: u16 }
```

---

**Question**: What do `clap` and `rayon` provide in the Rust ecosystem?

**Answer**: `clap` is a declarative CLI argument parser using builder API or derive macros (`#[derive(Parser)]`) — it handles subcommands, validation, help text, and shell completion generation. `rayon` provides parallel iterators via `par_iter()` and `par_bridge()`, automatically splitting work across threads using a work-stealing thread pool, with no manual thread management required.

```rust
use clap::Parser;
#[derive(Parser)]
struct Args { #[arg(short)] name: String }
```

---

**Question**: How do declarative macros (`macro_rules!`) differ from procedural macros (`#[derive]`, `#[proc_macro]`)?

**Answer**: `macro_rules!` uses pattern-matching on token trees for hygienic, compile-time code generation — ideal for repetitive boilerplate like `vec![]` or `println!()`. Procedural macros operate on the AST and run as compiler plugins, enabling `#[derive(Serialize)]` or attribute macros. Procedural macros are more powerful (arbitrary Rust code for transformation) but live in separate proc-macro crates.

```rust
// Declarative
macro_rules! double { ($x:expr) => { $x * 2 }; }
// Procedural (in proc-macro crate) — generates structured TokenStream → TokenStream
```

---

**Question**: What tooling commands are available in `cargo`, and how do profiles and conditional compilation work?

**Answer**: Key commands: `cargo build`, `cargo test`, `cargo run`, `cargo bench`, `cargo clippy` (lints), `cargo fmt` (formatting), `cargo doc` (API docs). Conditional compilation uses `#[cfg(...)]` attributes and `cfg!()` macros to include or exclude code by target OS, feature flag, or `cfg(test)`. Profiles in `Cargo.toml` (`[profile.release]`) control optimizations like `opt-level`, `codegen-units`, and LTO — setting `lto = true` and `codegen-units = 1` maximizes runtime performance at the cost of compile time.

```toml
[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
```

---

**Question**: How does `cargo doc` generate documentation and what conventions should documentation follow?

**Answer**: `cargo doc` compiles doc comments (`///` and `//!`) and runs embedded code examples as tests via `cargo test --doc`. Documentation should include a top-level module summary, per-item doc comments describing behavior and panic conditions, and runnable examples. Cross-crate links are resolved automatically using `[`Type`]` syntax for intra-doc links.
