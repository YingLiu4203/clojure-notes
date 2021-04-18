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

It is often advisable to have a dedicated server process running. `shadow-cljs server` runs the process in the foreground, use `Ctrl+C` to terminate it. `shadow-cljs clj-repl` let you control the server process in REPL.Use `shadow-cljs start/stop/restart` to control a background server.
