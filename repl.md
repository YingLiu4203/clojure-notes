# REPL

REPLE is a command-line interface to interact with a running Clojure program.

## Getting Around in REPLE

When you run `clj` from the command line, there are some commands available:

- `(source name)`: show the source code of a symbol.
- `(apropo "something")`: return a seq of all definitions (by name) that match the pattern.
- `(doc name)`: print documentation for a var, a special form or a namespace. For example: `(doc doc)`. `(doc str)`, `(doc clojure.repl)`.
- `(find-doc "text")` or `(find-doc #"(?i)text")`: find in the description for `text` or case-insenstive `text`.
- `(dir ns)`: show a list of functions and vars in a namespace.
- `(pst e)`: print exception mesage and stacktrace.
- `*1`, `*2`, `*3` are lat, second, and third most recent values.
- `*e` the last exception.
- `(javadoc name)`: show doc of a Java class.
- `(pprint expr)`: show a pretty formatted value.
- `(pp)`: pretty print the previous result, same as `(pprint *1)`
- History
  - Previous: Ctrl + P, Up arrow.
  - Next: Ctrl + N, Down arrow.
  - Search: Ctrl + R, press Ctrl + R to cycle through the matches, Enter to select one.
- Exit: type `Ctrl-D`.

## nREPL

`nREPL` stands for "network REPL". It is a message-based client-server REPL service. A client implements Read, Print, and Loop. The server handles the Evaluation and can do more such as looking up documentation, inspecting running program etc.
