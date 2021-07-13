# Namespace

## 1 Basics

Namespaces are global map objects that map var names (symbols) to var objects, or symbols to class objects. Not fully-qualified names are resovled to the current namespace. Var objects may have aliases. Compiled code uses fully-qualified references that multiple aliases point to the same var object.

### 1.1 Current Namespace

The current namespace is stored in the var `#'clojure.core/*ns*`. The `ns` macro or `in-ns` function change the thread-local binding value of `*ns*`. The global root of `*ns*` is `clojure.core`.

### 1.2 Auto-resolved Keyword

`::` is used to auto-resolve keyword in the current namespace. If no qualifier is specified, it will auto-resolve to the current namespace. If the keyword is qualified, the namespace will be resolved using aliases in the current namespace. For example, if the current namespace has an aliase `example` for `x`, then `::x/foo` is resovled as `:example/foo`. `::` is not available in edn.

### 1.3 Namespace Map

Namespace map is used to specify a default namespace context when keys or symbols in a map where they share a common namespace. The syntax is `#:ns{...}` that specifies a prfeix `ns`for all keys or symbols in a map.

```clojure
#:person{:first "Han"
         :last "Solo"
         :ship #:ship{:name "Millennium Falcon"
                      :model "YT-1300f light freighter"}}
; it is read as:
{:person/first "Han"
 :person/last "Solo"
 :person/ship {:ship/name "Millennium Falcon"
               :ship/model "YT-1300f light freighter"}}
```

Similarly, `#::{}` resolves the current namespace. `#::alias` works for namespace alias.

## 2 Using Namespaces

### 2.1 Creating Mappings Using `def`, `refer` and `import`

`def` creates a mapping from a symbol to a var in the current namespace.

Use `refer` to add mappings to the vars of a loaded namespace. By default it adds mappings of all vars of a namespace. You can specify the `:exclude`, `:only` and `:rename` to customize the mappings. `refer` is rarely used dirctly. Its effects and options are availabe through `use`.

Use `import` to map symbols to Java classes and interfaces. For example: `(import 'java.util.Date 'java.text.SimpleDateFormat)`. As `refer` or `use`, you can use a list to add mutiple mappings with a common prefix. For example: `(import '(java.util Arrays Collections))`.

### 2.2 `require`

`require` provides two functions:

- Load the specified namespaces
- Optionally create namespace aliases

In REPL, you need to quote a namespace after `require`. For example: `(require 'clojure.set)` or with an aliase: `(require '[clojure.set :as set])`.

When multiple namespaces share a common prefix, you can put namespaces in a list where the first element is the namespace prefix. For example: `(require '(clojure string [set :as set]))`.

### 2.3 `use`

`use` provides all functions of `require` and `refer`. For example:

```clojure
(use '(clojure [string :only (join) :as str]
               [set :exclude (join)]))
```

### 2.4 Get Info about Mappings, Aliases and Namespaces

- Use `ns-map` to find a map of all the mappigs for a namespace.
- Use `ns-aliases` to find all aliases of a namespace.
- Use `all-ns` to list all loaded namespaces.
- Use `find-ns` to search a namespace.

### 2.5 Problems with Namespace

- start time is slow: bootstraping `clojure.core` is slow.
- `Var.getRawRoot` overhead is high.
- Heap use. Namesapces are atomic. A basic REPLE uses over 17MB of heap.
- Impedes static analysis because a lot of indirections invocation.

## 3 Best Practices

It is good to always use `require` with an aliase for each namespace.

An alternate is to use `use` with a namesapce alias and `:only` to explicitly include a list of vars to `refer`.

Avoid cyclic namespace dependencies.

Start every namespace with a comprehensive `ns` form.

Use underscore in filename when namespaces contain dashes. For example `com.my-project.foo` is defined in `com/my_project/foo.clj`.

Use `declare` to enable forward references.

Avoid single-segement namespaces.
