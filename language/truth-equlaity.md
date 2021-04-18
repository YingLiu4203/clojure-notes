# Truth and Equality

## 1 Truth

### 1.1 Truthy and Falsy

`false` and `nil` are the only values that are treated as `falsy` in Clojure, everything else is `truthy`.

Use `true?`, `false?`, `nil?` to check exactly `true`, `false` or `nil` values. `=` can also be used.

### 1.2 Logic Opertors

`and` returns the first falsy value and skip the rest. When all are truthy, returns the last value.

`or` returns the first truthy value and skip the rest. When all are falsy, return the last value.

`not` returns `true` for falsy values and `false` for truthy values.

## 2 Equality

Clojure's equality operator `=` adopts the Java `equals` semantics. All sequential collections(vector, list or sequence) are equal if their elements are equal. Numbers in different types are not equal.

The numberical equivalence operator `==` returns `true` if the valuies represented by different numbers are numerically equivalent. This is the equivalence used by numeric keys in Clojure collection.

`(= (1 2) [1 2]) ;true`
`(= 1 1.0) ;false`
`(== 1 1.0) ;true`

Use `identical?` to check object identity.
