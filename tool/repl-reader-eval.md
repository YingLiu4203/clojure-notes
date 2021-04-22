# REPL, Reader and Evaluation

> > > Fundamentally, the reason programmers use the REPL for all these tasks is always the same: because they want a mix of automation and improvisation that can be provided neither by fully manual tools (such as dashboard, consoles, etc.) nor by fully automated ones (such as scripts), while keeping their workflow focused in one full-featured programming environmen. [Programming at the REPL](https://clojure.org/guides/repl/introduction)

## 1 Getting Started in REPL

REPL is a command-line interface to interact with a running Clojure program.

When you run `clj` from the command line, there are some commands (defined in [clojure.repl](https://clojure.github.io/clojure/clojure.repl-api.html). Use `(require '[clojure.repl :refer :all])` to make all commands avaialbel.

- `(source name)`: show the source code of a symbol.
- `(apropo "something")`: return a seq of all definitions (by name) that match the pattern.
- `(doc name)`: print documentation for a var, a special form or a namespace. For example: `(doc doc)`. `(doc str)`, `(doc clojure.repl)`.
- `(find-doc "text")` or `(find-doc #"(?i)text")`: find in the description for `text` or case-insenstive `text`.
- `(dir ns)`: show a list of functions and vars in a namespace.
- `(pst e)`: print exception mesage and stacktrace.

Special vars:

- `*1`, `*2`, `*3` are last, second, and third most recent values.
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

### 1.1 Namespace

- `*ns*`: current namespace.
- `ns foo`: create a new namespace and make it current.
- `in-ns bar`: switch to a new namespace.
- `require '[clojure.set :as cset :refere [union]]`: importe namespace and var.

### 1.2 nREPL

`nREPL` stands for "network REPL". It is a message-based client-server REPL service. A client implements Read, Print, and Loop. The server handles the Evaluation and can do more such as looking up documentation, inspecting running program etc.

### 1.3 REPL Phases

A REPL has the following phases:

- `:read-source`
- `:macro-syntax-check`
- `:macroexpansion`
- `:compile-syntax-check`
- `:compilation`
- `:execution`
- `:read-eval-result`
- `:print-eval-result`

## 2 Reader

Clojure is a homoiconic language that Clojure programs are represented by Clojure data structrues. Clojure is defined in terms of the evaluation of data structures and not in terms of the syntax of character streams/files. It is the task of the reader, defined by `read` function, to parse the text (forms represented by a sequence of characters) and produce the data structures (symbols, lists, vectors, maps etc.) the compiler will see.

The reader reads the following forms:

### 2.1 Symbols and Literals

- symbols: may be qualified by namespace or Java package. `.`, `/` and `:` have special meanings.
- literals: strings, numbers (`Long`, `BigInt`, `Double`, `BigDecimal`, ratio), characters (preceded by a backslash), `nil`, `true`, `false`, symbolic values (positive infinity `##Inf`, negative infinity`##-Inf`, and not-a-number `##NaN`), and keywords.

A keyword that begins with two colons is auto-resolved in the current namespace to a qualified keyword.

- If the keyword is unqualified, the namespace will be the current namespace. In user, `::rect` is read as `:user/rect`.
- If the keyword is qualified, the namespace will be resolved using aliases in the current namespace. In a namespace where `x` is aliased to `example`, `::x/foo` resolves to `:example/foo`.

### 2.2 Sequence or Special Forms

- Lists: forms enclosed in parentheses.
- Vectors: forms enclosed in square brackets.
- Maps: key/value paris enclosed in braces. May be qualified by a namespace `#:ns`. `#::` can be used to auto-resolve namespaces with the same semantics as auto-resolved keywords.
- Sets: forms enclosed in `#{}`.
- `deftype`, `defrecord` and constructor calls. They are called by fully qualified name preceded by `#` and followed by a vector or a map. For example, `#my.class[:a : c]` or `#my.record{:a 1, :b 2}`

### 2.3 Macro Characters

The behavior of the reader is driven by a combination of built-in constructs and an extension system called the `read table`. Entries in the read table provide mappings from certain characters, called `macro characters`, to specific reading behavior, called `reader macros`. Unless indicated otherwise, macro characters cannot be used in user symbols. You can think them as syntax sugars provided by the reader.

- Quote `'form` => `(quote form)`
- Character `\`: `\newline`, `\space`, `\tab`, `\formfeed`, `\backspace`, and `\return`. A Unicode literal is of the form `\uNNNN`.
- Comment `;`
- Defer `@form` => `(deref form)`
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
