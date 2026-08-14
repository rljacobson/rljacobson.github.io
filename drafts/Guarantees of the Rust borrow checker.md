# What makes Rust "safe"?

Generally, safety in Rust means: 

* Type safety
* Memory safety
* Thread safety
* Safety from undefined behavior / nasal demons

In particular, safety means:

* No use after free
* No dangling pointers
* No aliasing of mutable state
* No data races

Language features making Rust safe:

* Panics instead of undefined behavior
* Statically verified ownership and aliasing
* Strong type system



# Rust safety guarantees

Rust's safety guarantees are "soft"/"weak" guarantees, not mathematically rigorous guarantees.

| Description                                       | Guaranteed | Reasons                                                      |
| ------------------------------------------------- | ---------- | ------------------------------------------------------------ |
| Allocations are sound                             | No         | Allocations are unsafe code.                                 |
| Allocations are sound under assumption of no bugs | No         | The global allocator is unsafe by necessity.  "It’s undefined behavior if global allocators unwind.  … [C]urrently a panic ... may lead to memory unsafety." |
| `Panic`s are safe                                 |            | "Unless you are very careful and tightly control what code runs, pretty much everything can unwind, and you need to be ready for it." …"Code that transiently creates unsound states must be careful that a panic does not cause that state to be used." |
|                                                   |            |                                                              |
|                                                   |            |                                                              |
|                                                   |            |                                                              |
|                                                   |            |                                                              |
|                                                   |            |                                                              |
|                                                   |            |                                                              |

# The Rust compiler does not consider the following behaviors unsafe, though a programmer may (should) find them undesirable, unexpected, or erroneous.
The Rust compiler does not consider the following behaviors *unsafe*, though a programmer may (should) find them undesirable, unexpected, or erroneous.

* [Deadlocks](https://doc.rust-lang.org/reference/behavior-not-considered-unsafe.html#deadlocks)

* [Leaks of memory and other resources](https://doc.rust-lang.org/reference/behavior-not-considered-unsafe.html#leaks-of-memory-and-other-resources)

* [Exiting without calling destructors](https://doc.rust-lang.org/reference/behavior-not-considered-unsafe.html#exiting-without-calling-destructors)

* [Exposing randomized base addresses through pointer leaks](https://doc.rust-lang.org/reference/behavior-not-considered-unsafe.html#exposing-randomized-base-addresses-through-pointer-leaks)

* [Integer overflow](https://doc.rust-lang.org/reference/behavior-not-considered-unsafe.html#integer-overflow)

| Description                             |          | Explanation                                                  |
| --------------------------------------- | -------- | ------------------------------------------------------------ |
| No memory leaks                         | No       |                                                              |
| No race conditions                      | No       | Only prevents *data* races, not race conditions in general.  |
| `Send `/ `Sync` guarantee thread safety | Yes? No? | "In the *incredibly rare* case that a type is inappropriately automatically derived to be Send or Sync, then one can also unimplement Send and Sync." …"Note that *in and of itself* it is impossible to incorrectly derive Send and Sync. Only types that are ascribed special meaning by other unsafe code can possibly cause trouble by being incorrectly Send or Sync." |
|                                         |          |                                                              |
|                                         |          |                                                              |
|                                         |          |                                                              |
|                                         |          |                                                              |
|                                         |          |                                                              |
|                                         |          |                                                              |



# Unsafety of `unsafe`

Code within `unsafe` is unsafe in non obvious, subtle ways.

* "Unless you are very careful and tightly control what code runs, pretty much everything can unwind, and you need to be ready for it." …"Code that transiently creates unsound states must be careful that a panic does not cause that state to be used."





# Other Relevant Material

Computer Scientist proves safety claims of the programming language Rust
https://www.eurekalert.org/pub_releases/2021-07/su-cs071521.php

