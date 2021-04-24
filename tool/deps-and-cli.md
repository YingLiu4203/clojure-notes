# REPL, Deps and CLI

This article summarizes the Clojure REPL and CLI tools from the following resources:

- [The REPL and main](https://clojure.org/reference/repl_and_main)
- [REPL Introduction](https://clojure.org/guides/repl/introduction)
- [Deps and CLI](https://clojure.org/reference/deps_and_cli)
- [What are the Clojure tools](https://betweentwoparens.com/what-are-the-clojure-tools)

REPL is a command-line interface to interact with a running Clojure program.
Clojure CLI tools or `tools-deps` are just an umbrella term consisting of following parts to resolve/download dependendencis and build classpath to run a program in the JVM.

- `deps.edn` is an edn configuration file used to configure dependencies.
- `tools.deps.alpha` is a library that reads `deps.edn` and resolves dependency graphs and build classptaths that can utilize both Maven or other providers of code or artifacts. It doesn't provide build and project management.
- `clojure` or `clj` are bash scripts to run Clojure programs on the JVM. `clj` wraps `clojure` (another bash script) with `readline` support to make it easier to type manually in REPL. Under the hood, `clojure` uses `deps.edn` and `tools.deps.alpha`.
- System-specific installers for downloading the tools.

There are many [build or project management tools](https://github.com/clojure/tools.deps.alpha/wiki/Tools) based on the above CLI tools.

You install the tools via `brew install clojure/tools/clojure`. It shows as `clojure` in `brew info` after installation.

## 1 REPL

> > > Fundamentally, the reason programmers use the REPL for all these tasks is always the same: because they want a mix of automation and improvisation that can be provided neither by fully manual tools (such as dashboard, consoles, etc.) nor by fully automated ones (such as scripts), while keeping their workflow focused in one full-featured programming environmen. [Programming at the REPL](https://clojure.org/guides/repl/introduction)

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

## 2 CLI

Essentially the command is `java [java-opt*] -cp classpath clojure.main [clj-opt*] [main-opt] [arg*]`. `clj` starts a REPL using `clojure` that executes `clojure.main` to run a REPL or execute a function/script.

To execute a function, use `clojure [clj-opt*] -X[aliases] [a/fn] [kpath v]*`. The `-X` is configured with an arg map with `:exec-fn` and `:exec-args` keys. The map is stored as an alias in `deps.edn`. The `[kpath v]*` can be a single key or a vector of keys that is used to `assoc-in` to the `:exec-args` map.

For example, with the definition:

```clojure
{:aliases
 {:my-fn
  {:exec-fn my.qualified/fn
   :exec-args {:my {:data 123}
               :config 456}}}}
```

you can run `clj -X:my-fn :config 789` or `clj -X:my-fn '[:my :data]' 789`. The single quotes are required for strings, vectors, maps, sets and lists in command line. You can also execute a specific function direclty in command line such as `clj -X my.qualified/fn :config 789`.

Use `-M` exec-opt to invoke `clojure.main` that calls a namespace with `-main` function or a Clojure script. The command is `clojure [clj-opt*] -M[aliases] [main-opts]`.

The `clj-opt` could be:

- `-i, --init`: load a file or resource
- `e, --eval string`: evaluate expressions in string.
- `--report target`: report uncaught exception to `file`(default), `stderr` or `none`.
- `r, repl`: run a repl. If no options or args, this is the default.
- `path`: run a script from a file or resource
- `-m, --main`: a namesapce to find a `-main` function for execution
- `-h, --help`: print help message

To pass in arguments to a script, pass them in as futher argments when launching `clojure.main`. For example: `clj -M /path/to/script.clj arg1 arg2 arg3`. The arguments will be provided to your program as a seq of strings bound to the var `*command-line-args*`: `*command-line-args* => ("arg1" "arg2" "arg3")`.

## 3 Tool Execution

There are a series of steps performed to prepare and execute the Clojure tool.

### 3.1 Locate and configure with `deps.edn`

The Clojure tools look for 4 potential deps edn sources:

- Root - part of the clj installation (a resource in the `tools.deps` library).
- User - cross-project configuration (typically tools), usually found at `~/.clojure/deps.edn`.
- Project - the `deps.edn` in the current directory.
- Config data - a deps edn map passed on the command line.

### 3.2 Check Cache

Create classpathes and cache it. Use cache at `./.cpcache` or `~/.clojure` or `$CLJ_CACHE`/`$XDG_CACHE_HOME` first.

### 3.3 Replace project environment

When you execute a tool in the context of your project, the tool can use its own paths and/or deps in place of the project’s paths and/or deps.

### 3.4 Merge sources

All deps edn sources are merged into a single master edn file in the order listed above - root, user, project (possibly with tool replacements), config. This merged deps will be included in the runtime basis and used after this point.

The merge is essentially `merge-with merge`, except for the `:paths` key, where only the last :paths found is used (they replace, not combine).

### 3.5 Resolve Deps

Starting from the master edn’s merged :deps, the full transitive expansion of the dependency tree is computed. Dependency sources (procurers) are used to obtain metadata and other dependency information. At the completion of this step, all libraries to use in the execution have been found and downloaded to local files if needed.

### 3.6 Make classpath

JVM execution relies on computing the classpath for the execution. The classpath will consist of the :paths in the merged deps.edn and the locations of all resolved dependencies.

### 3.7 Prepare JVM environment

JVM arguments may either be passed on the command line (with `-J`) or by using data stored in an alias under :jvm-opts and passed with `-X` or `-A` or `-M`

### 3.8 Execute command

Finally the command is executed.

## 4 `deps.edn`

Edn maps has the following top-level keys:

- `:deps` - map of lib (symbol) to coordinate
- `:paths` - vector of project source paths
- `:aliases` - map of alias name to alias data
- `provider-specific` keys for configuring dependency sources

### 4.1 `:paths`

Paths are a vector of string paths or alias names. Relative paths are resolved in relation to the directory containing the deps.edn (the project root). These source paths will be added to the classpath. Source paths should be at or under the project root directory (where deps.edn is located). If used, alias names should refer to a path vector in the alias data.

### 4.2 `deps`

It is a map from library to coordinate. Libraries are symbols of the form `<groupID>/<artifactId>`. To indicate a Maven classifier, use `<groupId>/<artifactId>$<classifier>`. Coordinates can take several forms depending on the coordinate type:

- Maven coordinate: `{:mvn/version "1.2.3"}`
- Local project coordinate: `{:local/root "/path/to/project"}`
- Local jar coordinate: `{:local/root "/path/to/file.jar"}`
- Git coordinate: `{:git/url "https://github.com/user/project.git", :sha "sha", :tag "tag"}`

### 4.3 Aliases

Aliases give a name to a data structure that can be used either by the Clojure tool itself or other consumers of `deps.edn`.

- Basis and classpath: The core of the tools.deps library is a process to take a merged deps.edn file, arg maps for the resolve-deps and make-classpath steps, and produce the "runtime basis", or "basis" for short. The basis is a superset of the deps.edn file also containing those args, the lib map, and the classpath map. The process of building a classpath has two primary operations: `resolve-deps` and `make-classpath`.
  - The `resolve-deps` takes an initial map of required dependencies and a map of args that modify the resolution process.
  - The `make-classpath` step takes the lib map (the result of `resolve-deps`), the internal source paths of the project `["src"]`, an args-map of optional modifications, and produces a classpath string for use in the JVM.
    - `:extra-paths` - a collection of string paths to add to :paths (should be in the project)
    - `:classpath-overrides` - a map of lib to string path to replace the location of the lib
    - If multiple maps with these keys are activated, `:extra-paths` concatenate and `:classpath-overrides` merge-with merge.
- JVM enviornment: `:jvm-opts`
- Execution: `:main-opts`
