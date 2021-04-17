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

## Getting Started

### Installation

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

Create `deps.edn`:

```clojure
{:paths   ["src/main" "resources"]
 :deps    {org.clojure/clojure    {:mvn/version "1.10.3"}
           com.fulcrologic/fulcro {:mvn/version "3.4.21"}}

 :aliases {:dev {:extra-paths ["src/dev"]
                 :extra-deps  {org.clojure/clojurescript   {:mvn/version "1.10.844"}
                               thheller/shadow-cljs        {:mvn/version "2.12.4"}
                               binaryage/devtools          {:mvn/version "1.0.3"}}}}}
```
