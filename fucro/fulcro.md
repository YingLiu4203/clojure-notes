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

## 4 Feeding the Data Tree

### 4.1 Initial State

Fulcro allows you to co-locate the initial desired part of the data tree with the component that uses it. Fulcro pulls this data into the database when the application first mounts -- you need to reload browser if there is a change in initial state. The `defsc` macro takes an `{:initial-state (fn ...)}`. For example:

```clojure
(defsc Person [this {:person/keys [name age]}]
  {:initial-state (fn [{:keys [name age] :as params}] {:person/name name :person/age age}) }
  (dom/li
    (dom/h5 (str name "(age: " age ")"))))

(def ui-person (comp/factory Person {:keyfn :person/name}))

;; call the initial state function
(comp/get-initial-state Person {:name "Sally" :age 32})
```

In the above code, the value of `:initial-state` is a function that is called by `comp/get-initial-state Component params` in composition parent. `comp/get-initial-state` fetchs initial state given a component and parameters of initial state function. The initial state becomes part of initial application database. The creation and use only requires local reasoning.

Use `(com.fulcrologic.fulcro.application/current-state app.application/app)` ot check the current application state. It is available in the DB tab of Fulcro inspect dev tool.

### 4.2 Co-located Query

The queries on components describe what data the component wants from the database. The co-located query sets up data access for both the client and server with the benefit of local reasoning and composition. Like the UI and data, queries form a tree that starts with a join. A join is a map with a single entry whose key is the node keyword. Node data is represented as a vector that is relative to the current node. A map in a vector represents a join of a subquery and must have another vector to represent data of the subquery. For example:

```cloure
[{:friends  ; this is a JOIN
    [:list/label
      {:list/people ; JOIN
         [:person/name :person/age]}]}]
```

Joins are automatically to-one if the data found in the state is a singular, and to-many if the data found is a vector. It is recommended to use keyword with namespace.

In a component, the query is a vector that represent data to be feed by its parent. A parent calls `com/get-query` on it child components. For exmaple:

```clojure
;; query for Person
{:query [:person/name :person/age]}

;; query for PersonList
{:query [:list/label {:list/people (comp/get-query Person)}]}

;; for the root that has two PersonList
{:query [{:friends (comp/get-query PersonList)}
         {:enemies (comp/get-query PersonList)}]}
```

The queries are composed from bottom all the way to the root.

There is a global client database but each component queries only the local portion of its data. Local reasoning stays. The local database is stored in an atom. The application state can evolve consistently with the UI components.

### 4.3 Passing Callbacks to Child Components

A component definition in `defsc` has an additional paramter that you can desconstruct (or use `(comp/get-computed this)`) to use the callback passed from its parent. It is useful when a parent does something for its children. For example, a component composed as a list item has a delete button. It should call its parent to perform action because the component knows nothing about its parent.

To not skip the rendering of the parent, either use `comp/computed c {:key fn}` or `comp/computed-factory`.

### 4.4 Transaction

All changes to the client db should go through Fulcro transaction system. All changes are represented as operation data. You use namespaced operation name to identify an operation. It is like `` (comp/transact! this `[(ops/delete-person {:list-name ~name :person ~person})]) ``. The unquote is used to pass operation data to the transaction system.

The `defmutation` macro returns a function-like object that resturs itself as data. If you can require the mutation namespace without causing circular references, then you can just "call" the mutation within the transaction as-if it were a function (avoiding quoting). Otherwise, you must quote it. If the `api/delete-person` is defined with `defmutation`, it can be invoked as `(comp/transact! this [(api/delete-person {:list-name name :person person})])`. The call of `(api/delete-person {:list-name name :person person})` generates `(api/delete-person {:list-name ...})` with varirables replaced with their values.

A good practice is to put client mutation in `.cljs` file and server mutation in `.clj` file.

A mutation definition looks like a function. It can have a docstring and a single argument `params`. The `params` is a map that can be constructured. The body is like `letfn` with some pre-established names such as `action`. Below is an example:

```clojure
(ns app.mutations
  (:require [com.fulcrologic.fulcro.mutations :as m :refer [defmutation]]))

(defmutation delete-person
  "Mutation: Delete the person with `name` from the list with `list-name`"
  [{:keys [list-name name]}] ; the mutation arguments
  (action [{:keys [state]}] ...)) ; the action with the app-state atom as an argument
```

### 4.5 Data Normalization

In a graph db, a reference can have to-many arity. A foreign reference is represented as `[TABLE ID]`, it is called an `ident`. A table is an entity that is identified by a type-centric ID such as `:person/id` or `:account/id`. The following is a normalized graph db:

```clojure
;; The domain data
{:PersonList { :friends { :id :friends
                          :label "Friends"
                          :people #{1, 2} }}
 :Person { 1 {:id 1 :name "Joe" }
           2 {:id 2 :name "Sally"}}}

;; the Fandom graph data
{ :list/id
  { :friends
    { :list/id :friends
      :list/label "Friends"
      :list/people [[:person/id 1] [:person/id 2]]}}

  :person/id { 1 {:person/id 1 :person/name "Joe" }
               2 {:person/id 2 :person/name "Sally"}}}
```

In the graph data model, `:list/id` and `person/id` are used two times: one as the table name and one as a table property. Their values are repeated too: one as id for each entity and one as the entity's prop value.

To use the automatic normalization provided by Fulcro, you need to set `:ident` optioin for each component. It takes a function to get thte `ident` from its props. For example: `{:ident (fn [] [:person/id (:person/id props)]) }`

### 4.6 Mutation on a Normalized Db

With normalized db, you can reomve a foreign key from a Table entry, instead of removing an entity from a global tree. This is not only easier to code, but it is local: independent of the shape of the UI tree.

Fulcro’s `com.fulcrologic.fulcro.algorithms/merge` namespace includes tools for merging and managing normalized data and includes some helpers for dealing with lists of idents. Mutation "helpers" in fulcro are functions that work against a plain map (normalized app database) so that they can be used easily in `swap!`. By convention these functions have a `*` suffix, and often have mutation versions that do not have the suffix.

A mutation example is as below:

```clojure
(ns app.mutations
  (:require
    [com.fulcrologic.fulcro.mutations :as m :refer [defmutation]]
    [com.fulcrologic.fulcro.algorithms.merge :as merge]))

(defmutation delete-person
  "Mutation: Delete the person with `:person/id` from the list with `:list/id`"
  [{list-id   :list/id person-id :person/id}]
  (action [{:keys [state]}]
    (swap! state merge/remove-ident* [:person/id person-id] [:list/id list-id :list/people])))
```

The `remove-ident*` mutation helper does just that: It removes an ident from a list of idents. The arguments are the ident to remove and the path to the list of idents that you want to remove it from. Below is an example using the mutation:

```clojure
(defsc PersonList [this {:list/keys [id label people] :as props}] (1)
  ...
  (let [delete-person (fn [person-id]
    (comp/transact! this [(api/delete-person {:list/id id :person/id person-id})]))]
```

### 4.7 How Automatic Normalization Works

When you compose query via `comp/get-query`, for each component's query fragment, it adds the component name as the query meta data. For example `(meta (com.fulcrologic.fulcro.components/get-query app.ui/PersonList))` returns `{:component app.ui/PersonList, :queryid "app.ui/PersonList"}`.

Fulcro has a function called `com.fulcrologic.fulcro.algorithms.normalize/tree→db` that walks the data tree and queries together. When it reach a data node that matches a component's query and `ident` value, it replaces the data with its ident.
