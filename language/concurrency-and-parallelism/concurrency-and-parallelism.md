# Clojure Concurrency and Parallelism

## Concurrency

Java provides paltry few low-level resources such as threads and locks to deal with concurrency. Clojure provides three types of concurrency resources:

- immutable values: all basic data structures.
- reference types: `var`, `atom`, `ref`, `agent`.
- Clojure concurrency constructs: `delay`, `future`, `promise`.
- Java concurrency libs: thread, locks etc.

`clojure.lang.IDeref` defines the `deref` abstraction. All reference types and Clojure concurrency constructurs support this abstraction.

## Parallelism

When use parallel function like `pmap`, make sure that the operation on each value is significant enough to justify the parallelization overhead. Sometimes, chunking dataset helps.

`pcalls` and `pvalues` build on top of `pmap`. `pcalls` takes any number of no-arg functions and returns a lazy sequence. `pvalues` does the same but for any number of expressions.

## State and Identity

Clojure distinguishes state and identity. An identity can have a particular state at any point in time. A state is an immutable value. Clojure uses four reference types: `var`, `atom`, `ref` and `agent` as identities. All references contain some values acceesible by `deref` or `@`.

Dereferencing returns a snapshot of the state. Unlike `delay`, `future` and `promise`, dereferencing a reference type will never block and will never interfere with other operations.

All reference types:

- may be decorated with metatdata that can only be changed by `alter-meta!` function.
- Can notify functions using `watch`.
- Can enforce constraints using `validator`.

Classifying reference types:

- `ref`: coordinated, sync
- `atom`: uncoordinated, sync
- `agent`: uncoordinated, async

The coordinated + async combination is more common in distributed systems. Clojure focuses in in-process concurrency.
