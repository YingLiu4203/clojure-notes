# Destructuring

Destructuring has two main data types: sequential data (a list or a vector, but not set) and associative data (a map). Destructuring can be used in `let` and function parameters.

To destruct a sequence, use `[[e1 e2 e3] col]`.

To destruct a map, use `[{n1 :k1 n2 :k2 n3 :k3} dic]`. If the symbols and keys have the smae name, use the short syntax: `[{keys: [k1 k2 k3]} dic]`.

- Use `_` to discard a destructured value.
- Use `& xs` to put the rest into a sequence represented by the `xs`.
- Desctructuring supports nested structures.
- You can have multiple destructuring and a destructured value can be used in later steps.
