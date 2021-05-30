# Clojure Concurrency

Java provides paltry few low-level resources such as threads and locks to deal with concurrency. Clojure provides three types of concurrency resources:

- immutable values: all basic data structures.
- reference types: `var`, `atom`, `ref`, `agent`.
- Clojure concurrency constructs: `delay`, `future`, `promise`.
- Java concurrency libs: thread, locks etc.

`clojure.lang.IDeref` defines the `deref` abstraction. All reference types and Clojure concurrency constructurs support this abstraction.
