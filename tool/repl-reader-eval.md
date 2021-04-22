# REPL, Reader and Evaluation

> > > Fundamentally, the reason programmers use the REPL for all these tasks is always the same: because they want a mix of automation and improvisation that can be provided neither by fully manual tools (such as dashboard, consoles, etc.) nor by fully automated ones (such as scripts), while keeping their workflow focused in one full-featured programming environmen. [Programming at the REPL](https://clojure.org/guides/repl/introduction)

## 1 REPL

REPL is a command-line interface to interact with a running Clojure program. Resources:

- [The REPL and main](https://clojure.org/reference/repl_and_main)
- [REPL Introduction](https://clojure.org/guides/repl/introduction)

### 1.1 Launching

- You run `clj` from the command line that invokes `clojure.main`.
- To run a file full of Clojure code as a script, pass the path to the script to `clojure.main` as an argument: `clj -M /path/to/myscript.clj`.
- To pass in arguments to a script, pass them in as further arguments like `clj -M /path/to/myscript.clj arg1 arg2 arg3`. The arguments will be provided to your program as a seq of strings bound to the var `*command-line-args*`.

### 1.2 Tool Commands

There are some tool commands (defined in [clojure.repl](https://clojure.github.io/clojure/clojure.repl-api.html) available when you use `(require '[clojure.repl :refer :all])` to make them avaialbel.

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

### 1.3 History avagiation

- Previous: Ctrl + P, Up arrow.
- Next: Ctrl + N, Down arrow.
- Search: Ctrl + R, press Ctrl + R to cycle through the matches, Enter to select one.

To Exit, type `Ctrl-D`.

### 1.4 Namespace

- `*ns*`: current namespace.
- `ns foo`: create a new namespace and make it current.
- `in-ns bar`: switch to a new namespace.
- `require '[clojure.set :as cset :refere [union]]`: importe namespace and var.

### 1.5 nREPL

`nREPL` stands for "network REPL". It is a message-based client-server REPL service. A client implements Read, Print, and Loop. The server handles the Evaluation and can do more such as looking up documentation, inspecting running program etc.

### 1.6 REPL Phases

A REPL has the following phases:

- `:read-source`
- `:macro-syntax-check`
- `:macroexpansion`
- `:compile-syntax-check`
- `:compilation`
- `:execution`
- `:read-eval-result`
- `:print-eval-result`

## 2 [Reader](https://clojure.org/reference/reader)

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

### 2.3 Reader Macro Characters

The behavior of the reader is driven by a combination of built-in constructs and an extension system called the `read table`. Entries in the read table provide mappings from certain characters, called `macro characters`, to specific reading behavior, called `reader macros`. Unless indicated otherwise, macro characters cannot be used in user symbols. You can think them as syntax sugar provided by the reader.

- Quote `'form` => `(quote form)`
- Character `\`: `\newline`, `\space`, `\tab`, `\formfeed`, `\backspace`, and `\return`. A Unicode literal is of the form `\uNNNN`.
- Comment `;`
- Defer `@form` => `(deref form)`
- Metadata `^` -- a map like `^{:dynamic true}` with shorthand `^:dynamic` or a type tag like `^{:tag java.lang.String` with shorthand `^String`.
- Dispatch `#` causes the reader to use a reader macro from another table, indexded by the character following:
  - set `#{}`
  - regex `#"pattern"`: read and complied at read time with a result object of type `java.util.regex.Pattern`.
  - var-quote `#'` => `(var x)`
  - anonymous function, `#()` => `(fn [args] (...))`. It cannot be nested.
  - ignore next form `#_`
- Syntax quote (backtick), unquote (`~`) and unquote-splicing (`~@)`
  - For all forms other than symbols, lists, vectors, sets and maps, backtick is the same as quote.
  - For Symbols, syntax-quote resolves the symbol in the current context, yielding a fully-qualified symbol.
  - If a symbol is non-namespace-qualified and ends with `#`, it is resolved to a generated symbol with the same name to which `_` and a unique id have been appended.
  - For Lists/Vectors/Sets/Maps, syntax-quote establishes a template of the corresponding data structure. Within the template, unqualified forms behave as if recursively syntax-quoted, but forms can be exempted from such recursive quoting by qualifying them with unquote or unquote-splicing, in which case they will be treated as expressions and be replaced in the template by their value, or sequence of values, respectively.

There are tagged literals that are parsed by data readers.

Clojure 1.7 introduces `reader conditions` using`.cljc` files for portable code that cna be loaded by multiple platforms. Reader conditionals are a new reader dispatch form starting with `#?` or `#?@`. Both consist of a series of alternating features and expressions, similar to cond. Every Clojure platform has a well-known "platform feature" - `:clj`, `:cljs`, `:cljr`. there is a `:default` keyword for unspecified platforms.

### 2.4 Extensible Data Notation (`edn`)

[`edn`](https://github.com/edn-format/edn) is used as a data transfer format. It is a system for the conveyance of values, not a type system, has no schemas, not representing objects. All values are immutable.

It supports a rich set of built-in elements including `nil`, booleans, strings, characters, symbols, keywords, integers, floats, lists, vectors, maps, and sets.

`edn` supports extensibility through tagged elements. `#` followed by a symbol starting with an alphabetic character indicates that that symbol is a `tag`. The reader dispatchs the data to regitered client handler.

Tag symbols without a prefix are reserved by `edn` for built-ins defined using the tag system. For example: `#inst "1985-04-12T23:20:50.52Z"` for an instant time, `#uuid "f81d4fae-7dec-11d0-a765-00a0c91e6bf6"` for a UUID.

User tags must contain a prefix component, which must be owned by the user (e.g. trademark or domain) or known unique in the communication context.

`;` the line is a comment. `#_` discard the next element.

### 2.5 Tagged Literals

Tagged literals are Clojure's implementation of end tagged elements. When Clojure starts, it searches for files named data_readers.clj at the root of the classpath. Each such file must contain a Clojure map of symbols. The key is a tag and the value is the fully-qualified name of a `Var` which will be invoked by the reader to parse the form following the tag.

Reader tags without namespace qualifiers are reserved for Clojure. Default reader tags are defined in `default-data-readers` but may be overridden in `data_readers.clj` or by rebinding `*data-readers*`. If no data reader is found for a tag, the function bound in `*default-data-reader-fn*` will be invoked with the tag and value to produce a value. If `*default-data-reader-fn*` is `nil` (the default), a `RuntimeException` will be thrown.

### 2.6 Reader Conditionals

Clojure 1.7 introduced a new extension `.cljc` for portable files that can be loaded by multiple Clojure platforms.

Reader conditionals are a new reader dispatch form starting with `#?` or `#?@`. Both consist of a series of alternating features and expressions, similar to `cond`.

Every Clojure platform has a well-known "platform feature": `:clj`, `:cljs`, `:cljr`. A well-known `:default` feature will always match and can be used to provide a default. If no branches match, no form will be read (as if no reader conditional expression was present).

The reader conditional will read and return that feature’s expression. The syntax for `#?@` is exactly the same but the expression is expected to return a collection that can be spliced into the surrounding context, similar to unquote-splicing in syntax quote. Use of reader conditional splicing at the top level is not supported and will throw an exception. For example: `[1 2 #?@(:clj [3 4] :cljs [5 6])]`.

## 3 Evaluation

### 3.1 Introduction

Evaluation can occur in many contexts:

- Interactively, in the REPL
- On a sequence of forms read from a stream, via `load` / `load-file` / `load-reader` / `load-string`.
- Programmatically, via `eval`.

Clojure programs are composed of expressions that have the following structures:

- macros
- forms
  - special forms
  - expressions

Only `expressions` in the above are evaluated to yield values. There are no declarations or statements, although sometimes expressions may be evaluated for their side-effects and their values ignored. If an expression needs to be compiled, it will be. There is no separate compilation step, nor any need to worry that a function you have defined is being interpreted. Clojure has no interpreter.

### 3.2 Evaluation Rules

- Strings, numbers, characters, true, false, nil and keywords evaluate to themselves.
- A symbol is resolved.
  - A namespace-qualified symbol is resolved to its binding value.
  - A packaged-qualified symbol is resolved to its Java class.
  - A special form is executed.
  - A local symbol is resolved to its local binding value.
  - A symbol is resolved to current namespace for Java class or a var.
- Vectors, Sets and Maps yield vectors, sets and maps whose content are evaluated values of the objects they contain. Vector elements are evaluated left to right, Sets and Maps are evaluated in an undefined order.
- Non-empty Lists are considered calls to either special forms, macros, or functions. A call has the form `(operator operands*)`. An empty list `()` is an empty list.
  - Special forms are primitives built-in to Clojure that perform core operations.
  - Macros are functions that manipulate forms, allowing for syntactic abstraction.
  - If the operator is not a special form or macro, the call is considered a function call. Both the operator and the operands (if any) are evaluated, from left to right. The result of the evaluation of the operator is cast to `IFn` (the interface representing Clojure functions), and `invoke()` is called on it, passing the evaluated arguments. The return value of `invoke()` is the value of the call expression.
- Any object other than those discussed above will evaluate to itself.

If a Symbol has metadata, it may be used by the compiler, but will not be part of the resulting value.
