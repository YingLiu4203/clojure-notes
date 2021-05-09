# Special Forms and Flow Control

Resources:

- [Special Forms](https://clojure.org/reference/special_forms)
- [Flow Control](https://clojure.org/guides/learn/flow)

## 1 Special Forms

Special forms have evaluation rules that differ from standard Clojure evaluation rules and are understood directly by the Clojure compiler. In special forms are not evaluated by Clojure's `eval` function. The following is a list of 13 special forms:

- `(def symblo doc-string? init?)`: creates and interns a global `var` with the name of `symblol` in current namespace. Many macros such as `defn` and `defmacro` expand into `def`.
- `(if test then else?)`: if `test` is not `nil` or `false`, evaluates and yields `then`, otherwise, evaluate and yields `else`. `else` defaults to `nil`. Only Java's singular value `Boolean.FALSE` is treated as `false`.
- `(do expr*)`: evaluates expresions `exprs` in order and returns the value of the last. No expression returns `nil`.
- `(let [binding*] expr*)`: The `binding` is `binding-form init-expr`. The `bindings` are sequential, so each `binding` can see the prior bindings. The `exprs` are contained in an implicit `do`.
- `(quote form)`: yiedls the unevluated `form`. Same as reader macro `'`.
- `(var symbol)`: the `symbol` must resolve to a var, and the `Var` object itself, not its value, is returned. Same as reader macro `#'`.
- `(fn name? [params* ] expr*)` or `(fn name? ([params* ] expr*)+)`: Defines a function (`fn`). `Fns` are first-class objects that implement the `IFn` interface. The `IFn` interface defines an `invoke()` function that is overloaded with arity ranging from 0-20.
  - The exprs are `enclosed` in an implicit `do`.
  - If a `name` symbol is provided, it is bound within the function definition to the function object itself, allowing for self-calling, even in anonymous functions.
  - If a `param` symbol is annotated with a metadata tag, the compiler will try to resolve the tag to a class name and presume that type in subsequent references to the binding.
  - A `fn` (overload) defines a recursion point at the top of the function, with arity equal to the number of params including the rest param, if present.
  - Functions support specifying runtime pre- and post-conditions as `{:pre [pre-expr*] :post [post-expr*]}`.
- `(loop [binding* ] expr*)`: `loop` is exactly like `let`, except that it establishes a recursion point at the top of the `loop`, with arity equal to the number of bindings.
- `(recur expr*)`: Evaluates the expressions `exprs` in order, then, in parallel, rebinds the bindings of the recursion point to the values of the `exprs`. It re-uses the one stack frame and can only happen at the tail position.
- `(throw expr)`: throw a `Throwable`.
- `(try expr* catch-clause* finally-clause?)`: same as Java's `try/catch/finally`.
- `.`, `new`, `set!`: for Java interop. `set!` can also set a `Var`.
- Binding Forms (Destructuring): In `let` binding lists, `fn` parameter lists, destructuring creates a seet of bindings to values within a collection: a `vector` form specifies bindings by position in a sequential collection (any thing supports `nth`), a `map` form by key in an associative collection.

## Flow Control

Clojure's special forms, `if`, `do`, and `recur` (with `loop` or `fn`) define basic controls of condition, sequence and loop. Clojure provides more flow controls via the following 7 macros.

- `when`: an `if` with only a `then` branch.
- `cond` and `else`: a series of tests and expressions. The expression of the first true test returns.
- `case`: compare an argument to a series of values to find a match. Throw an `IllegalArgumentException` if no value matches. A final trailing expression can be used as the default if no test matches.
- `dotimes`: evaluate expression `n` times.
- `doseq`: iterates over a sequence and return `nil`. It can have multiple bindings.
- `for`: is a list comprehension. Bindings behave like `doseq`.
- `with-open`: calls the `.close` method of the binding variable.
