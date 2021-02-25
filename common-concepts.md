# Common Concepts

## Destructuring

Destructuring has two main data types: sequential data (a list or a vector, but not set) and associative data (a map). Destructuring can be used in `let` and function parameters.

To destruct a sequence, use `[[e1 e2 e3] col]`.

To destruct a map, use associative destructuring such as `[{n1 :k1 n2 :k2 n3 :k3} dic]`. If the symbols and keys have the same name, use the short syntax: `[{keys: [k1 k2 k3]} dic]` or `[{keys: [:k1 :k2 :k3]} dic]

- Use `_` to discard a destructured value.
- Use `& xs` to put the rest into an `ArraySeq` represented by the `xs`.
- Desctructuring supports nested structures.
- You can have multiple destructuring and a destructured value can be used in later steps.

## `apply`

Apply can take a function and one or more single or collection arguments. It may be helpful to assign a default value in case the collection is empty and the function does'n handle the emepty collection.

## Threading Macros

The threading expressions are function calls of the form `(f arg1 arg2 …​)`.

- Thread-first `->`: put the initial value as the first argument of the first expression. Then put each expression result as the first argument of the next expression.
- Thread-last `->>`: put the initial value as the last argument of the first expression. Then put each expression result as the last argument of the next expression.
- Thread-as `as->`: bind value to a symbol and use the symbol in the following expressions.

A bare symbol or keyword without parentheses is interpreted as a simple function invocation with a single argument. This allows for a succinct chain of unary functions.

Optionally, you can use three commas to mark the place where the argument will be inserted.

```clojure
(-> person :hair-color name clojure.string/upper-case)
;; equivalent to
(-> person (:hair-color) (name) (clojure.string/upper-case))

(->> [range 10]
    (filter odd? ,,,))

(as-> [:foo :bar] v
  (map name v)
  (first v)
  (.substring v 1))
```

`some->`, `some->>` are used in Java interop and stops when a `nil` value occurs.

The macro `cond->` and `cond->>` take an initial value and interpret their argument list as a series of test, expr pairs. `cond->` threads a value through the expressions but skips those with failing tests.
