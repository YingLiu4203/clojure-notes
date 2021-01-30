# REPL, Reader and Evaluation

## Getting Around in REPLE

REPL is a command-line interface to interact with a running Clojure program. The [REPL resources page](https://clojure.org/guides/repl/annex_community_resources) has links to videos and tutorials to REPL.

When you run `clj` from the command line, there are some commands (defined in [clojure.repl](https://clojure.github.io/clojure/clojure.repl-api.html). Use `(require '[clojure.repl :refer :all])` to make all commands avaialbel.

- `(source name)`: show the source code of a symbol.
- `(apropo "something")`: return a seq of all definitions (by name) that match the pattern.
- `(doc name)`: print documentation for a var, a special form or a namespace. For example: `(doc doc)`. `(doc str)`, `(doc clojure.repl)`.
- `(find-doc "text")` or `(find-doc #"(?i)text")`: find in the description for `text` or case-insenstive `text`.
- `(dir ns)`: show a list of functions and vars in a namespace.
- `(pst e)`: print exception mesage and stacktrace.

Special vars:

- `*1`, `*2`, `*3` are lat, second, and third most recent values.
- `*e` the last exception.

Additional commands from `clojure.java.javadoc` and `clojure.pprint`:

- `(javadoc name)`: show doc of a Java class.
- `(pprint expr)`: show a pretty formatted value.
- `(pp)`: pretty print the previous result, same as `(pprint *1)`

History avagiation

- Previous: Ctrl + P, Up arrow.
- Next: Ctrl + N, Down arrow.
- Search: Ctrl + R, press Ctrl + R to cycle through the matches, Enter to select one.

To Exit, type `Ctrl-D`.

## Namespace

- `*ns*`: current namespace.
- `ns foo`: create a new namespace and make it current.
- `in-ns bar`: switch to a new namespace.
- `require '[clojure.set :as cset :refere [uniion]]`: importe namespace and var.

## nREPL

`nREPL` stands for "network REPL". It is a message-based client-server REPL service. A client implements Read, Print, and Loop. The server handles the Evaluation and can do more such as looking up documentation, inspecting running program etc.

## REPL Phases

A REPL has the following phases:

- `:read-source`
- `:macro-syntax-check`
- `:macroexpansion`
- `:compile-syntax-check`
- `:compilation`
- `:execution`
- `:read-eval-result`
- `:print-eval-result`

## Reader

Clojure is defined in terms of the evaluation of data structures and not in terms of the syntax of character streams/files. It is the task of the reader, defined by `read` function, to parse the text and produce the data structure the compiler will see.

The text has the following forms:

- symbols: may be qualified by namespace or Java package
- literals: strings, numbers, characters, `nil`, `true`, `false`, symbolic values (`##Inf`, `##-Inf`, and `##NaN`), keywords.
- Lists: forms enclosed in parentheses.
- Vectors: forms enclosed in square brackets.
- Maps: key/value paris enclosed in braces. May be qualified by a namespace `#:ns`.
- Sets: forms enclosed in `#{}`.
- `deftype`, `defrecord` and constructor calls.

The behavior of the reader is driven by macro characters tool

- Quote `'`
- Character `\`
- Comment `;`
- Defer `@`
- Metadata `^` -- use a map or type shortcut `^String` and dynamic shortcut `^:dynamic`
- Dispatch `#`: use for set, regex, var-quote `#'`, anonymous function, `#()`, ignore next form `#_`.
- Syntax quote: the backquote character, unquote `~`, and unquote-splicing `~@`.

There are tagged literals that are parsed by data readers.

Clojure 1.7 introduces `reader conditions` using`.cljc` files for portable code that cna be loaded by multiple platforms. Reader conditionals are a new reader dispatch form starting with `#?` or `#?@`. Both consist of a series of alternating features and expressions, similar to cond. Every Clojure platform has a well-known "platform feature" - `:clj`, `:cljs`, `:cljr`. there is a `:default` keyword for unspecified platforms.

## Evaluation

Evaluation can occur in many contexts:

- Interactively, in the REPL
- On a sequence of forms read from a stream, via `load` / `load-file` / `load-reader` / `load-string`.
- Programmatically, via `eval`.

Clojure programs are composed of the following:

- macros
- forms
  - special forms
  - expressions

The evaluation rules:

- Strings, numbers, characters, true, false, nil and keywords evaluate to themselves.
- A symbol is resolved.
  - A namespace-qualified symbol is resolved to its binding value.
  - A packaged-qualified symbol is resolved to its Java class.
  - A special form is executed.
  - A local symbol is resolved to its local binding value.
  - A symbol is resolved to current namespace for Java class or a var.
- Vectors, Sets and Maps yield vectors, sets and maps whose content are evaluated values of the objects they contain. Vector elements are evaluated left to right, Sets and Maps are evaluated in an undefined order.
- An empty list `()` is an empty list.
