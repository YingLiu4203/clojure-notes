# Fulcro

This is a study note of [Fulcro](https://book.fulcrologic.com/).

## 1 Overivew

### 1.1 From 10,000 feets

Fulcro is a full-stack application programming system that is graph-centric and uses CQRS architecture style.

- Data models are graphs.
- UI trees are graphs that can be fed from graph queries.
- User operations are command data.
- Graph data are normalized with a centralized managing/caching/trasanction processing systems.
- UI components are state mahcines that support actor model.
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

In Fulcro, the UI pulls data from client database and render the components. The mutations evovle the database to a new version.
