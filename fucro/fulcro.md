# Fulcro

This is a study note of [Fulcro](https://book.fulcrologic.com/).

## 1 Overivew

### 1.1 From 10,000 feets

Fulcro is a full-stack application programming system that is graph-centric and uses CQRS architecture style.

- Data models are graphs.
- Graph data are normalized with a centralized managing/caching/trasanction processing systems.
- UI trees are graphs of UI components that are functions transforming the graph database into the UI.
- UI components are state mahcines that support actor model.
- UI component operations are mutation data (in a CQRS style) that are processed by transact systems for local db, remote service and error handling.
- Subscription model is simply a query or a push with the backend from this centralized client database.

Fulcro is written in CLJC thus is able to setup and run in both JS or JVM. It uses [EQL: EDN Query Language](https://edn-query-language.org/) to query data in a declarative way. It uses the [pathom](https://github.com/wilkerlucio/pathom) library to process EQL requests and reshape any particular schema into the required graph model for clients.

Fulcro uses a single, central, normalized graph database that fullfills all UI queries. Graph nodes are stored in tables. Edges are represented as ventors that have a format of `[TABLE ID]`. The term `ident` means a tuple of table and an ID `[:TABLE id]` that uniquely identifies a graph node.

A Fulcro component defines the query, the ident and the DOM component. You can put the component data into the database or load data from the database.

UI components submit transactions as mutations. Mutations are data that describs many actions: local and optimistic changes of client databse, remote server interactions, handling of network error/normal results etc.

Thus in Fulcro, data flows in and out via graph queries and mutations. It manages a local normalized graph database and interact with remote server using a CQRS-style transactional semantic.

### 1.2 Core Concepts

- immutable data structure: safe, time-travel, efficient change detection.
- pure rendering: simple logic.
- data driven: for both query and mutation.
- client-side graph database: a simple, persistent, graph map.
  - Items are joined into a graph using an `ident`: a tuple of a table name and a key. For example `:person/by-id 4]`. `ident` is the same used in query and mutation of the db.
  - An `ident` appears in a vector meaning a `to-many` relation.
  - Graph can have loops.
  - The db drives UI.

Naming conventions to avoid collisions in db:

- UI only props: `:ui/name`.
- Tables: `:entity-type/index-indicator`. For example: `:person/id`.
- Root props: `:root/prop-name`
- Node props: `:entity-type/property-name`. For example: `:person/name` or `:graph/data`.
- Singleton components: consistent ident for singleton UI elements. For example: `[:component/id ::Component]` where `::Component` is the keywrodized version of the fully-qualified component name.

In Fulcro, the UI pulls data from client database and render the components. The mutations evovle the database to a new version.

## 2 Getting Started

### 2.1 Installation

Dev tools include Fulcro inspect extensiion for Chrome, Binaryage dev lib.
Chrome setting: Console -> Enable custom formatters, Network -> Disable Cache (when Devtools is open).

Fulcro template contains boilerplate on things such as server configuration, CSRF, testing, better error message formatting, etc.

Install JDK, Clojure CLI tools and Node.

```zsh
mkdir app
cd app
mkdir -p src/main src/dev resources/public
npm init
#... answer the questions or just take defaults ...
npm install shadow-cljs react react-dom --save
```

### 2.2 Configuration

Create `deps.edn` for Clojure dependencies:

```clojure
{:paths   ["src/main" "resources"]
 :deps    {org.clojure/clojure    {:mvn/version "1.10.3"}
           com.fulcrologic/fulcro {:mvn/version "3.4.21"}}

 :aliases {:dev {:extra-paths ["src/dev"]
                 :extra-deps  {org.clojure/clojurescript   {:mvn/version "1.10.844"}
                               thheller/shadow-cljs        {:mvn/version "2.12.4"}
                               binaryage/devtools          {:mvn/version "1.0.3"}}}}}
```

Add a `shadow-cljs.edn` for ClojureScript:

```clojure
{:deps     {:aliases [:dev]}
 :dev-http {8000 "classpath:public"}
 :builds   {:main {:target     :browser
                   :output-dir "resources/public/js/main"
                   :asset-path "/js/main"
                   :modules    {:main {:init-fn app.client/init
                                       :entries [app.client]}}
                   :devtools   {:after-load app.client/refresh
                                :preloads   [com.fulcrologic.fulcro.inspect.preload]}}}}
```

- The top-level `dev-http` server will cause shadow to start a dev web server on a specified port and serves files from the specified directory. In this case, `resources` is in class path defined in `deps.edn` and the folder is `resources/public` directory.
- The `:modules` section configures what code gets pulled into a given build.
- The `:entries` are entry-point namespaces that shadow will foullow requires to build the project.
- The `init-fn` are code to setup application on load.
- The `after-load` are code to trigger UI refreshs on hot code reload.

### 2.3 HTML Root File

Creat `resources/public/index.html` as the following:

```html
<html>
  <meta charset="utf-8" />
  <body>
    <div id="app"></div>
    <script src="/js/main/main.js"></script>
  </body>
</html>
```

### 2.4 Application Code

Create `src/main/app/client.cljs` as below:

```clojure
(ns app.client
  (:require
    [com.fulcrologic.fulcro.application :as app]
    [com.fulcrologic.fulcro.components :as comp :refer [defsc]]
    [com.fulcrologic.fulcro.dom :as dom]))

(defonce app (app/fulcro-app))

(defsc Root [this props]
  (dom/div "TODO"))

(defn ^:export init
  "Shadow-cljs sets this up to be our entry-point function. See shadow-cljs.edn `:init-fn` in the modules of the main build."
  []
  (app/mount! app Root "app")
  (js/console.log "Loaded"))

(defn ^:export refresh
  "During development, shadow-cljs will call this on every hot reload of source. See shadow-cljs.edn"
  []
  ;; re-mounting will cause forced UI refresh, update internals, etc.
  (app/mount! app Root "app")
  ;; As of Fulcro 3.3.0, this addition will help with stale queries when using dynamic routing:
  (comp/refresh-dynamic-queries! app)
  (js/console.log "Hot reload"))
```

### 2.5 Build and Run the App

Run the command `npx shadow-cljs watch main` and you should see HTML page at `http://localhost:8000`.

The `shadow-cljs` server runs on port `9630` and the nREPL runs on `58403`. You can use VS code to connect to this nREPL. You can connect to the `shadow-cljs` nREPL or application nREPL. The `nREPL` button at the bottom left allows you to switch.

## 3 UI Component

The `defsc` macro create React components. Use `{:use-hook? true}` to create functional components. These components are augmented with Fulcro's data management and render refresh.

### 3.1 Render

The `com.fulcrologic.fulcro.dom` has factoryfunctions to create DOM elements. The following is a template:

```clojure
(defsc ComponentName
  [this props] ; parameters. Available in body, and in *some* of the options
  ; optional:  { ...options... }
  (dom/div {:className "a" :id "id" :style {:color "red"}}
    (dom/p "Hello")))
```

The DOM element functions allow class names and IDs to be written in various forms for convenience. The following are all equivalent:

```clojure
(dom/div {:id "id" :className "x y z" :style {:color "red"}} ...)
(dom/div :.x#id {:className "y z" :style {:color "red"}} ...)
(dom/div :.x.y.z#id {:style {:color "red"}} ...)
(dom/div :.x#id {:classes ["y" "z"] :style {:color "red"}} ...)
```

If you’re writing your UI in CLJC files then you need to make sure you use a conditional reader to pull in the proper server DOM functions for Clojure:

```clojure
(ns app.client
  (:require #?(:clj [com.fulcrologic.fulcro.dom-server :as dom]
               :cljs [com.fulcrologic.fulcro.dom :as dom]))
;;... the actual code is the same as befor
```

### 3.2 Props

In Fulcro you will usually use props for most data. You can get props by calling `comp/props` on this, or by destructuring in the second argument of `defsc`.

### 3.3 Component Factory

To use a component, you need a component factory, the convention is to use `ui-` prefix for the factory. Then use the factory to create a component. For example:

```clojure
; a component with props
(defsc Person [this {:person/keys [name age]}]
  (dom/div
    (dom/p "Name: " name)
    (dom/p "Age: " age)))

; factory
(def ui-person (comp/factory Person))

; use
(defsc Root [this props]
  (dom/div
    (ui-person {:person/name "Joe" :person/age 22})))
```
