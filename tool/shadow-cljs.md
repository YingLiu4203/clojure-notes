# Shadow CLJS

Resources:

- [What Shadow-cljs is and isn't](https://code.thheller.com/blog/shadow-cljs/2019/03/01/what-shadow-cljs-is-and-isnt.html)
- [User Guide](https://shadow-cljs.github.io/docs/UsersGuide.html)
- [Repository](https://github.com/thheller/shadow-cljs)
- [Blogs](https://code.thheller.com/)

## 1 What is shadow-cljs?

`shadow-cljs` is a fully featured build tool for ClojureScript and JavaScript. It integrates with the `npm` ecosystme and allows accessing it from ClojureScript. It runs on the JVM and uses the `Closure Compiler` to process JS and create optimized JS output. It runs on JVM.

A build process has 4 stages: setup, compile, organize, optimize.

- Setup: `shadow-cljs` initializes and validates the setup for compiler. It extracts classpath from `deps.cljs` and create a `:foreign-lib` index. It also builds an index of `namespace -> file` from Closure libs and from npm libs (`:npm-deps`). At the end of setup, compiler states are bound to the `cljs.env/*compiler*` atom and compilation starts.
- Compile ClojureScript: `tools.reader`, `cljs.analyzer` and `cljs.compiler` read `.cljs` source, analyze code, expand macros and generate JS code with source maps. Every CLJS namespace has an implicit dependency on `cljs.core` and all dependences are analyzed first before compilation.
- Organize JS: add or generate required JS files and put all output to targets.
- Optimize: this is only done for release build. `:optimizations :none` is set for development and in REPL. Closure compiler performs optimiation in this step.

`shadow-cljs` performs setup, organize and optimize, using ClojureScript to compile the code with customization for `npm` integrtion. Most work are done at the organize and optimize steps that both deal with JS code only.

There are some conflicts in resolving JS `:npm-deps` regarding `.`, `/`, `@` and `-` interpretation because of different meanings in CLJS and JS. JS doesn't have a proper namespace system, using only relate file paths. Allowing strings in `:require` sovles this issue.

## 2 Introduction

`shadow-cljs` has two parts: a Clojure lib handles actual work and an npm package named `shadow-cljs` that provides CLI commands. You define one or more builds in the `shadow-cljs.edn` configuration file. Each build will have a `:target` property which represents a configuration preset optimized for the target environment such as a Browser or Node.js. The standard build commands are: `compile`, `watch` and `release`.

### 2.1 The Classpath

`shadow-cljs` use JVM and its classpath when work with files. It is based on the model of a "virtual filesystem" that consists of many classpath entries that are created from two sources:

- A local filesystem directory defined by `:source-paths` entry in the configuration.
- A `.jar` file, representing a Clojure(Script) or JVM lib, added by `:dependencies`.

Everything is namespace and proper namespacing is recommended: `your-company.module.name` is preferred over `module.name`.

### 2.2 Server Mode and REPL

`shadow-cljs` can be started in "server" mode which is required for long-running tasks such as `watch`. The server is shared by all CLI commands and requires restart when `:dependencies` change.

The `watch` option enables hot-reload and provides a REPL.

You should use commands from REPL instead of CLI.

It is often advisable to have a dedicated server process running. `shadow-cljs server` runs the process in the foreground, use `Ctrl+C` to terminate it. `shadow-cljs clj-repl` let you control the server process in REPL. Use `shadow-cljs start/stop/restart` to control a background server.

### 2.3 `tools.deps`

To use `tools.deps` manage `:dependencies` and `:source-paths`, first add `shadwo-cljs` to `deps.edn`: `:deps {thheller/shadow-cljs {:mvn/version <latest>}}`. Then set `:deps true` or uses an `:cljs` alias in `shadow-cljs.edn` file.

## 3 Configuration

The configuration is defined in `shadow-cljs.edn` file in project root directory. You can use `shadow-cljs init` to create a default one.

### 3.1 Source and Deps Configuration

- Source Paths: `:source-paths` configures your JVM classpath. The compiler will use this config to find Clojure(Script) source files (eg. `.cljs`)
- Java Dependencies: `:dependencies` is a vector that each entry is a vector using `lib-name "version-string"]` syntax.
- To activate additional aliase defined in `deps.edn`, use `:deps` and `:aliases`. For example: `{:deps true}` or `{:deps {:aliases [:cljs]}}`.
- REPL: `:nrepl` is used to configure port, middleware etc for the nREPL server. `:socket-repl` for a socket REPL.
- HTTP server: `:http` to configure port and host for primary HTTP server. `:dev-http` for development server.
- JVM options: `:jvm-opts` to configure JVM parameters.

### 3.2 Build Configuration

Each build in `:builds` describes artifacts that the compiler will build. Each build defines a `:target`. Each target may have `:dev` and `:release` modes. Common targets include `:browser`, `:node-script`, `:node-library`, `:browser-test`, `:karma`, and `:react-native` etc.

`:devtools` has ocnfiguration for `:repl-init-ns`, `:repl-pprint`, `:preloads` etc.

To use hot code reload, run `shadow-cljs watch build-id`.

You can set metadata on normal CLJS `defn` vars to inform the compiler that these functions should be called in live reloading: `^:dev/before-load`, `^:dev/after-load`. These can be configued using `:before-load`, `:before-load-async`, `:after-load`, `:after-load-async` etc.

Use `:build-hooks` to define hook functions.

Use `:compiler-options` for compiler options. For example, `{:compiler-options {:warnings-as-errors #{:undeclared-var}}}`.

`:build-options {:cache-level :off}` to control cache. Levels are `:all`, `:jars` and `:off`.

### 3.3 Other configurations

- A restricted set of config options can be added to `~/.shadow-cljs/config.edn` which will then apply to all projects built on this users machine.
- You can declare npm dependencies directly by including a `deps.cljs` with `:npm-deps` in your project (eg. `src/main/deps.cljs`). You can also provide extra `:foreign-libs` definitions here. They won’t affect shadow-cljs but might help other tools.
