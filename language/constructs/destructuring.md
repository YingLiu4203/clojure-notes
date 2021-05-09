# Destructuring

[Destructuring](https://clojure.org/guides/destructuring) has two types: sequential destructuring (for a list or a vector or a sequential type such as as string, but not set) and associative destructuring (for an associative structure such as a map, a record or a vector etc). Destructuring can be used in `let` and function parameters.

To destruct a sequence, use `[[e1 e2 e3] col]`.

To destruct a map, use associative destructuring such as `[{n1 :k1 n2 :k2 n3 :k3} dic]`. If the symbols and keys have the same name, use the short syntax: `[{keys: [k1 k2 k3]} dic]` or `[{keys: [:k1 :k2 :k3]} dic]

- Use `_` to discard a destructured value.
- Use `& xs` to put the rest into an `ArraySeq` represented by the `xs`.
- Use `:as all-var` to bind the entire sequence to a vriable.
- Use `:or {var default-value}` to supply a default vvalue if the key is not present.
- Desctructuring supports nested structures.
- You can have multiple destructuring and a destructured value can be used in later steps.

Use `:keys` for associative values with keyword keys, use `:strs` for string keys and `:syms` for symbol keys. It works for function calls with keyword arguments. Use `:ns/keys` to bind namespaced keywords.
