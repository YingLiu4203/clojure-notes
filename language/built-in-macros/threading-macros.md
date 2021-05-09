# Threading Macros

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
