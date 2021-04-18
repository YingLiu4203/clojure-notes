# Deps and CLI

Resources:

- [The REPL and main entry points](https://clojure.org/reference/repl_and_main)
- [Deps and CLI](https://clojure.org/reference/deps_and_cli)

## 1 Overview

Clojure runs on JVM that has two capabilities:

- the `classpath` to invoke JVM processes and/or `URLClassLoader`s.
- transitive dependency download and resolution from Maven repositories.

Clojure provides

- `tools.deps.alpha` is a library for resolving dependency graphs and building classptaths that can utilize both Maven or other providers of code or artifacts. It doesn't provide building and project management.
- CLI tools `clojure` and `clj` use the capability to resolve dependencies and run programs.
- System-specific installers for downloading the tools.

You use `clojure` or `clj` to run Clojure programs on the JVM. `clj` ues `clojure` but has extra support for use as a REPL. They define classpaths, execution environment, the `main` class, ang args. The `deps.edn` file tells Clojure the source code path and what libs you use. You often use maps to define aliases that are keywords that name data structures.

## 2 CLI

Both `clj` and `clojure` are CLI tools that take `[clj-opt*]` and `[exec-opt]`. `clj` starts a REPL using `clojure` that executes `clojure.main` to run a REPL or execute a function/script.

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
