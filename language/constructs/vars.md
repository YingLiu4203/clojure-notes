# Vars

Resources:

- [Vars and Global Environment](https://clojure.org/reference/vars)

## Basics

Top level functions and valuies are all stored in vars. The compiler attempts to resolve all free symbols as `Vars`. `Vars` provide a mechanism to refer to a mutable storage location that can be dynamically rebound on a per-thread basis. Each `Var` can (but needn't) have a root binding that is shared by all threads that don't have a per-thread binding. Thus, the value of a `Var` is the value of its per-thread binding, or, if it is not bound in the thread requesting the value, the value of the root binding, if any.

The `var` special form or the `#'` reader macro can be used to get an interned `Var` object instead of its current value. It is mainly used in conjunction with `meta`.

The special form `def` creates (and interns) a `Var`. If the `Var` did not already exist and no initial value is supplied, the var is unbound. Supplying an initial value binds the root (even if it was already bound).

The Namespace system maintains global maps of symbols to `Var` objects. If a `def` expression does not find an interned entry in the current namespace for the symbol being def-ed, it creates one, otherwise it uses the existing `Var`. This find-or-create process is called `interning`.

It is possible to create vars that are not interned by using `with-local-vars`. These vars will not be found during free symbol resolution, and their values have to be accessed manually. But they can serve as useful thread-local mutable cells.

## Defining Vars

`def` always defines top level vars that are globally accessible.

- private vars: defined with `^:private`. It can only be referred by fully qualified name and can only be accessed by manually dereferencing `@#'name`. `defn-` defines a private function.
- docstrings: immediately after the name, can be accessed by `doc name`.
- consts: `^:const` a permanent value at compile time.
- dynamic: `^:dynamic` allows per-thread binding.

Defining a var without a value creates an `unbunded` var. It is often used as a `forward declaration`. Use `declare` macro for this purpose.

For mutable data, define a var to hold a reference type value and use `swap!`, `alter!`, `send`, and `send-off` etc to modify the data.

## Binding and Assignment

By default Vars are static. You can change the root value using `(alter-var-root #'name alter-fn)`.

Vars can be marked as dynamic (`^:dynamic`) to allow per-thread bindings via the macro binding. Bindings created with `binding` is a per-thread binding that cannot be seen by any other thread.

Clojure compiler emits different code with using a dynamic `Var`:

- Static vars are dereferenced using `Var.getRawRoot()`. It uses root binding.
- Dynamic vars are defreferenced using `Var.get()`. It uses thread-local binding and is slow.

There are scenarios that one might wish to redefine static Vars within a context and Clojure provides the functions `with-redefs` and `with-redefs-fn` for such purposes.

Functions defined with `defn` are stored in Vars, allowing for the re-definition of functions in a running program via `def` or `defn` the name again.

The Clojure-native concurrency functions of `future`, `send`, `send-off` and `pmap` support binding conveyance: the current dynamic bindings are conveyed to another thread. Binding convenyance doesn't hold for lazy seqs (as in `map`) in general.

The assignment special form `(set! var-symbol expr)` sets the current thread binding of a global var. You cannot assign to function params or local bindings. Only Java fields, Vars, Refs and Agents are mutable in Clojure.

## Related Functions

- Variants of `def`: `defn` `defn-` `definline` `defmacro` `defmethod` `defmulti` `defonce` `defstruct`
- Working with interned Vars: `declare` `intern` `binding` `find-var` `var`
- Working with Var objects: `with-local-vars` `var-get` `var-set` `alter-var-root` `var?` `with-redefs` `with-redefs-fn`
- Var validators: `set-validator!` `get-validator`
- Common Var metadata: `doc` `find-doc` `test`
