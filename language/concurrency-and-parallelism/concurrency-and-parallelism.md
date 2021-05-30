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
