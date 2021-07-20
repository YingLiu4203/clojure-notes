# Destructuring

Resource:

- [Destructuring](https://clojure.org/guides/destructuring)
- [Clojure's Destructuring](https://danielgregoire.dev/posts/2021-06-13-code-observation-clojure-destructuring/)

## 1 Introduction

Destructuring has two types: sequential destructuring (for a list or a vector or a sequential type such as as string, but not set) and associative destructuring (for an associative structure such as a map, a record or a vector etc).

To destruct a sequence, use `[[e1 e2 e3] col]`.

To destruct a map, use associative destructuring such as `[{n1 :k1 n2 :k2 n3 :k3} dic]`. If the symbols and keys have the same name, use the short syntax: `[{keys: [k1 k2 k3]} dic]` or `[{keys: [:k1 :k2 :k3]} dic]

- Use `_` to discard a destructured value.
- Use `& xs` to put the rest into an `ArraySeq` represented by the `xs`.
- Use `:as all-var` to bind the entire sequence to a vriable.
- Use `:or {var default-value}` to supply a default vvalue if the key is not present.
- Desctructuring supports nested structures.
- You can have multiple destructuring and a destructured value can be used in later steps.

Use `:keys` for associative values with keyword keys, use `:strs` for string keys and `:syms` for symbol keys. It works for function calls with keyword arguments. Use `:ns/keys` to bind namespaced keywords.

## 2 Sequential

### 2.1 Single Item Retrival

- Positional: `first`, `fnext`, `last`, `nth`, `rand-nth`, `second`
- Positional for nested sequences: `ffirst`
- Specific to certain data structures: `aget`, `peek`
- Based on truthiness: `some`, `when-first`
- A Clojure set, when invoked, returns the value supplied if it's contained in the set, else `nil`

In destruturing, the position is buind to the value of `nth coll pos`. If `pos` is beyond the boundary, return `nil`.

`nth` is implemented by `Indexed`, `CharSequence`, `Array`, `RandomAccess`, `Matcher`, `Map.Entry`, and `Sequential`.

### 2.2 Sub-sequence Retrival

- Positional: `butlast`, `drop`, `drop-last`, `next`, `nnext`, `random-sample`, `rest`, `split-at`, `subs`, `subvec`, `take`, `take-last`, `take-nth`
- Positional for nested sequences: `nfirst`, `nthnext`, `nthrest`
- Specific to certain data structures: `pop`
- Based on truthiness: `drop-while`, `split-with`, `take-while`

use `[_1 _2 & ?rest]` to match extra arguments. A corresponding number of calls to `first` and `next` are used to establish the positional and final `?rest` bindings.

## 3 Associative Data Structure

When a key is not a keyword, string, or symbol, then we have access to another form of associative destructuring. For example: `(let [{api-id "code"} card] api-id)`. For maps, we have to be explicit about the key whose value we want to bind to a symbol, in this case `"code"`.

Associative destructuring is powered by `get`, both for retrieving values from maps and providing defaults via `:or`.

Because get accepts nearly every type of data and returns nil by default, you can put almost anything on the right-hand side of an associative destructuring expression.

## 4 Keyword Arguments: When Sequences become Maps

You can write Clojure functions that take required positional arguments followed by optional "keyword arguments". Inserting a `&` marks the beginning of variable arguments in a function definition, and if those variable arguments are of an even number, you can treat that sequence as a map when destructuring.

Starting with Clojure version 1.11.0-alpha, callers of functions that accept keyword arguments can provide either an even number of values (as described above) or a single map.

The destructuring works in function definition and in `let`.

```clojure
(defn f [a b & {:keys [alpha]}]
  alpha)

(=
 (f nil nil :alpha 42)

 (let [[& {:keys [alpha]}] [:alpha 42]]
   alpha)

 42)
```

## 5 Destructuring Context

Destructuring can be used in `let` and function parameters in `fn` and `defn`.

The `destructure` function takes a vector of bindings and compiles them such that destructuring expressions are expanded: `(destructure '[[?] (1 2 3)])`

Both `let` and `loop` are defined using the `destructure` function directly, and any macro that uses either of those to establish bindings provides end-user support for destructuring. For example, `for` support desctructuring of its left-hand bindings, i.e., each individual item from the right-hand collection:

```clojure
(def coll (map (comp char (partial + 65)) (range 26)))
(for [[?1 ?2 ?3] [coll (next coll)]]
  [?1 ?2 ?3])
;;=> ([\A \B \C] [\B \C \D])
```
