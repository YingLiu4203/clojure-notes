# Macros

Clojure has a `reader`, a `macro expander` and a `compiler`. A Clojure program has multiple phases.

- reading: the `reader` converts a string into Clojure data structures. Reader macros are expanded.
- Macro expansion: macros are expanded recursively untill all forms are fixed. Both macro and reader macros are expaned.
- Compilation: the data sturctures are compiled into host code.
- Evaluation: execute host code. Clojure is a dynamic language. At runtime, `eval` can read, expand, compile and evaluate arbitrary code.
