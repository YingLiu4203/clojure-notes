# Abstractions

- [Multimethods and Hierarchies](https://clojure.org/reference/multimethods)
- [Datatypes](https://clojure.org/reference/datatypes)

## 1 Overview

Clojure is based on abstractions and provides tools to create abstractions. Here the abstraction is defined as a set of operations and data types implement abstractions. Clojure's design philosophy could be described as "it is better to have 100 functions to operate on one abstraction than 10 functions on 10 abstractions".

The main way to achieve abstraction is by polymorphism: an operation name is associated to different data types or different algorithms.Clojure let you dispatch an operation based on either a type or a dispatch function. Type-dispatch is more efficient but function-dispatch is dynamic and more flexible.

The `multimethod`, `protocol` and datatype features of `deftype`, `defrecord` and `reify` provide mechanisms for abstraction and data structure definition.

## 2 Built-in Abstractions and Types

Clojure has 7 built-in collection abstractoins: `collection`, `sequence`, `associative`, `indexed`, `stack`, `set`, and `sorted`. These abstractions are specified by host interfaces.

There are predicates to check the interface and type: `coll?`, `associative?`, `indexed?`, `sorted?`, `seq?`, `list?`, `vector?`, `set?`, and `map?`.

The built-in implemenation of 4 basic data structure types `list`, `vector`, `map`, and `set` has two distinctive characteristics:

- The are first and foremost used in terms of abstractions, not the concrete types.
- They are immutable and persistent.

Each data strurctue has multiple concrete implementations. It is a common practice that Clojure builds a rich set of auxiliary functions base on a a small set of essential APIs. The built-in abstractions such as sequencs, collections, callability etc, are specified by host interfaces, and the implementations by host classes.

### 2.1 Collection

All concrete implementations of the four basic data structures supports the functions defined in the `collection` abstraction:

- `conj`: add an item to a collection.
- `count`: get the number of items in a collection.
- `seq`: get a sequence of a collection, either a `nil` or a sequence with at least one element.
- `empty`: obtain an empty instance of the same type as a provided collection.
- `=`: determine the value equality of a collection compared to one or more other collections. `(= '(1 2) [1 2])` is `true`.

`clojure.core` provides many fundamental operations based on the `collection` abstraction, such as: `into`, `map`, `filter`, `remove`, `take`, and `drop` etc.

Each function is polymorphic and may be optimized for each concrete type. For example, `conj` prepends items to a `list` but appends lists to a `vector`.

The `empty` works as an identity function that create an empty instance for the specified collection data.

### 2.2 Sequences

The `sequence` abstract defines following functions in addition to the base provided by the collection abstraction:

- `first`: get the first element. Retrns a `nil` for a `nil` or an empty sequence.
- `rest`: get a sequence of all but the first element. For `nil` or empty sequence, retrn an empty sequence.
- `cons`: prepdens an element to collection and returns a sequence.

There are two commonly used operations:

- `next`: get a sequence of all but the first element. Retrns a `nil` for a `nil` or an empty sequence.
- `lazy-seq`: produces a lazy sequence that is the result of evaluating an expression.

The set of types that are `sequable` include: all Clojure collection types, all Java collections (`java.util.*`), all Java maps, all `java.lang.CharSequences` inclduing `String`, any type implmenting `java.lang.Iterable`, arrays, `nil`, any type implementing `clojure.lang.Sequable`.

There are three ways to create a `sequence`: `(seq col)`, `(cons e col)` and `(list* arg1 arg2 ... col)`(create a sequence from a number of elements and a collection).

Functions like `map` or `filter` return “lazy” `seqs`. Some functions like `range`, `vals`, or `keys` return their own specific type of `seq`.

Clojure's `list` implements the `sequence` abstraction but `sequence` and `list` are different in some important ways:

- Getting the length of a `seq` is costly. `list` keeps its lenght.
- The contents of sequences can be cmoputed lazily and can be infinite.

For a complete list of sequence operations, check the [seuqence reference doc](https://clojure.org/reference/sequences). There are operations in the following categories:

- Seq in, seq out: shorter, longer, missing tails, nested seqs, transforming
- Using a seq: extract, construct, compute a boolean, fore evaluation, check evaluation
- Creating a seq: lazy seq from collection/producer funciton/constant/other objects.

### 2.3 Associative

An `associative` abstraction defines four functions:

- `assoc`: create a new association from a collection.
- `dissoc`: drop associations for given keys from the collection.
- `get`: looks up the value for a key. Return `nil` if ther is no entry -- use `contains?` to resolve confusion.
- `contains?`: a predicate of whether a key exists.

A `vector` is an association whose keys are indexes of its value positions.

Common map functions

- Create a new map: `hash-map` `sorted-map` `sorted-map-by`
- 'change' a map: `assoc` `dissoc` `select-keys` `merge` `merge-with` `zipmap`
- Examine a map: `get` `contains?` `find` `keys` `vals` `map?`
- Examine a map entry: `key` `val`

### 2.4 Indexed, Stack, Set, Sorted

- Indexed: defines `nth` function.
- Stack: defines `conj`, `pop`, `peek`.
- Set: Set can be treat as a sort of degenerated map, associating keys with themselves. It defines a `disj` function. Higher level operations such as `difference`, `union`, `intersection` etc are described in [Set API doc](https://clojure.github.io/clojure/clojure.set-api.html).
- Sorted: only `map` and `set` have sorted variants. They support operatoins such as
  - `rseq`: reversed sequence
  - `subseq`: a sub sequence with specified ranged of keys.
  - `rsubseq`: revsed `subseq`.

## 3 Multimethods

### 3.1 What is a Multimethod

Clojure's `multimethod` system supports runtime polymorphism through dispatching on types, values, attributes and metadata of, and relationships between, one or more arguments. It has a dispachting function defined by `defmulti` macro and one or more `methods` defined by `defmethod` macro.

Multimethod is more flexible than the Java-style single dispatch. Both Clojure protocol and Java use the single dispatch based on the class/type of the first argument. The multimethod allows customized dispatch function based on properties of all arguments and an ad hoc hierarchy.

Additionally, the `remove-method` function removes a method assoicated with a dispatch-value and the `prefer-method` funtion creates an ordering to sovle dispatching ambiguities.

### 3.2 When Should You Use Multimethods

Essentially the `multimethod` solving the `branch-and-call` cases where you need to add code in two places (a branch decision and a callee function) to add a new feature. Simple brach-by-type cases are addressed by protocols while more complicated cases such as dispatch-by-multiple-arguments or ad hoc type or type-derivation can be address by multimethods. Multimethods are very flexible but rare in practice. The easist kind is dispatch-by-class where classes are dynamically defined by a dispatch function. The general rules are:

- If a function branches based on a type or multiple types, consider a multimethod.
- Types do not have to be explicit Java classes or data tag. They can be domain-specific.
- You should be able to interpret the dispatch value of a `defmethod` without having to refere to the `defmulti`.
- Don't use multimethods to handle optinal arguments (simple `nil` case) or recursion.

### 3.3 Value Match and Derivation

Multimethods dispatch is based on derivation and uses `isa?` rather than `=` when testing for dispatch value matches. By default, the hierarchy is globalled defined. Use `make-hierarchy` to create a hierarch object that can be use as a frist argurment of the above functions.

Derivation is determined by a combination of either Java inheritance (for class values), or using Clojure’s ad hoc hierarchy system. The hierarchy system supports derivation relationships between names (either symbols or keywords), and relationships between classes and names. The `derive` function creates these relationships, and the `parents`, `ancestors` and `isa?` functions query the hierarchy.

## 4 Protocols

### 4.1 Motivations

- Provide a high-performance, dynamic polymorphism construct for Clojure.
- Work as host's inteface: specification only (no implementation), a type can implement multiple protocols.
- Avoid interface's drawback: can be extended independently, not an `isa` relationship, no hierarchy.
- Support the 90% case of multimethods because dispatching by type is the most common use case. Each multimethod defines one operation, a protocol can include a set of related operations.

### 4.2 Basics

A protocol is a named set of named methods and their signatures, defined using `defprotocol` and has the following features:

- There are no implementations.
- It yields a set of polymorphic functions that dispatch on the type of the first argument.
- It is dynamic and doesn't require AOT compilation.
- It creates a host's interface with the same fully qualified name.
- You can implement a protocol on `nil`.

The `deftype`, `defrecord` or `reify` support protocol directly.

Use `extend` to extend a type to implement protocols using `protocl + function map` pairs. `extend-type` and `extend-protocol` are convenience macros.

If a protocol ahs `:extend-via-metadata true`, then values can extend the protocol by adding metadata where keys are fully-qualified protocol function symbols and values are function implementations.

Protocol implementations are checked first for direct definitions (`defrecord`, `deftype`, `reify`), then metatdata defintions, then external extensions (`extend`, `extend-type`, `extend-protocol`).
