# Macros

A macro is a special function that transforms Clojure data structure: the input is data structure and the output is data structure. A function has a metadata `:macro true` and is evaluated during compilation.

At a high level, Clojure program has four phases.

- reading: the `reader` converts a string into Clojure data structures. Reader macros are expanded.
- Macro expansion: macros are evaluated/expanded recursively. Its output is sent back to reading phase untill all forms are fixed.
- Host code generation: the data sturctures are compiled into host code.
- Evaluation: execute host code. Clojure is a dynamic language. At runtime, `eval` can read, expand, compile and evaluate arbitrary code.

Together, the macro expansion and host code generation form the compilation of Clojure source code. The two tricky facts about Clojure program execution are:

- a macro is a function executed in compilation and generates a textual data structure.
- the generates data structures is the input to reading process until no more macro expansion required.
- the `eval` at runtime involves all four phases.
