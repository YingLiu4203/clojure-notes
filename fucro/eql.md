# EQL

A noted based on two videos [The Maximal Graph](https://youtu.be/IS3i3DTUnAI) and [Spice up your life with eql](https://youtu.be/UvJEBMOtayk) and [EQL Documentation](https://edn-query-language.org/).

## 1 Motivation

In a typical SSR, an url like `customers/2` is sent to the server. The server parses the input, fetches the data and render the HTML. There are two things involved: `which` and `what`. Which is the reference to one or many data. What is the data to be retrieved and may have three forms: stored data, computed data and related entities. For a data API, the client may request different `what`. The REST API complects `which` and `what` thus is not a good fit. A client should be able to specify which and what recursively. The query syntax should be descriptive and independent of implementations. GraphQL is an example that has a custom language and typed schema.

GraphQL doesn't fit Clojure because Clojure is property-based, not type-based. EDN query language (EQL) is created for Clojure.

## 2 Overivew

To enable `AAA`(anybody can say anything about any topic), it uses namespaces. For example: `mybank.customer/id`.

- EQL is just Clojure EDN data enclosed in `[]`.
- The data is a vector of properties expressed as keywords. For example:`[:math/pi]`
- It uses a map to express a join that referes a related data. For example: `[{:github/viewer [:github.user/login]}]`.
- It supports parameters enclosed in `()`. For example: `[{(:github.user/gists {:github/first 10})}]`.
- It support mutations.
- It doesn't support enums, variables, fragments and directives.
- You can use specs to define types and enums. Use `def` and `defn` for variables, fragments and directives.

- Pathom use resolves to represent edges on the graph and connect properties. Resolvers are maps.
- Resolvers enable contextual auto-complete inference. If the contextual is given, there is no need to call the parent resolver.

## 3 Design of Pathom and EQL

The design of Pathom is based on four principles:

- Immutability: `[parse ctx path]`. Context feeds data to a path.
- EDN: `[parse ctx eql]`. Use edn-based eql.
- Context-free: use namespaced keyword. For example: `:video/id` and `:youtube.video/id`.
- Flat: the context-free ids allow data flatting. You can bring in any to-one reference at the same level of the data.

To-many references needs join. It also defines a scope. For example:

```clojure
;; resolver by a specific id
(defresolver video-comments
  {:input #{:youtube.video/id}
   :output [{:youtube.video/title
             :youtube.video/comments
             [:youtube.comment/body
              :youtube.comment.author/title]}]}
  (fn [env input] ...))

;; query by id
[{[:youtube.video/id "..."]
 [:youtube.video/title]}]

;; resolver by a global identifier, no :input
(defresolve favorite-videos
  {:output [{:favorite-videos [:youtube.video/id]}]}
  (fn [env input] ...))

;; a query with parameters -- can be access in env
[{(:favorite-videos {:limite 5 :search "Clojure"})
 [:youtube.video/title]}]

;; a mutation definition
(defmutation `new-user
  {:params [:user/name :user/email]
   :output [:user/id]}
   (fn [env attrs] ...))

;; a mutation command and its return result: read after mutation.
[{(new-user {:user/name "Bob" :user/email "bob@e.com"})
 [:user/id :user/name :user/email]}]

```

There is no context, no data type. All needed is a video id. There are two types of root scope: a specified id like `{[:youtube.video/id "..."] [:youtue.video/title]}`, or a singleton/global identifier such as `{:favorite-videos [:youtube.video/title]}`.

EQL supports mutations using `()` and symbols.

Pathom knows the graphs and can minimize API calls and handle concurrency. Additionally, the id aliase allow integration of multiple data services.

## 4 Specification

### 4.1 Query / Transactions

An EQL transaction is represented by an EDN vector. The program format is an AST defined by a map. An empty query is as the following:

```clojure
[] ; empty transaction

; ast
{:type :root, :children []}
```

Properties are expressed as keywords, either simple or qualified.

```clojure
[:album/name :album/year]

; ast
{:type     :root
 :children [{:type :prop, :dispatch-key :album/name, :key :album/name}
            {:type :prop, :dispatch-key :album/year, :key :album/year}]}
```

Joins are used to describe nested query. They are EDN maps that always has a single entry. The entry key is the property to join on, and the entry value is a sub-query to run.

```clojure
; join
[{:favorite-albums
  [:album/name :album/year]}]

; join ast
{:type     :root
 :children [{:type         :join
             :dispatch-key :favorite-albums
             :key          :favorite-albums
             :query        [:album/name :album/year]
             :children     [{:type :prop, :dispatch-key :album/name, :key :album/name}
                            {:type :prop, :dispatch-key :album/year, :key :album/year}]}]}
; nested join
[{:favorite-albums
  [:album/name :album/year
   {:album/tracks
    [:track/name
     :track/duration]}]}]

; nest join ast
{:type :root
 :children
 [{:type         :join
   :dispatch-key :favorite-albums
   :key          :favorite-albums

   :query        [:album/name
                  :album/year
                  {:album/tracks [:track/name :track/duration]}]

   :children     [{:type :prop, :dispatch-key :album/name, :key :album/name}
                  {:type :prop, :dispatch-key :album/year, :key :album/year}
                  {:type         :join
                   :dispatch-key :album/tracks
                   :key          :album/tracks
                   :query        [:track/name :track/duration]
                   :children     [{:type :prop, :dispatch-key :track/name, :key :track/name}
                                  {:type         :prop
                                   :dispatch-key :track/duration
                                   :key          :track/duration}]}]}]}
```

Idents are represented by a vector with two elements: `[keyword anything]`. The second `anything` works like a lookup reference that dependes on implementation. It is common to use an ident as a join key to start a query for some entity.

```clojure
; ident
[[:customer/id 123]]

; ast, the :dispatch-key and :key are different
{:type :root
 :children [{:type :prop, :dispatch-key :customer/id, :key [:customer/id 123]}]}

; ident as a join key
[{[:customer/id 123]
  [:customer/name :customer/email]}]

; ast
{:type     :root
 :children [{:type         :join
             :dispatch-key :customer/id
             :key          [:customer/id 123]
             :query        [:customer/name :customer/email]
             :children     [{:type :prop, :dispatch-key :customer/name, :key :customer/name}
                            {:type :prop, :dispatch-key :customer/email, :key :customer/email}]}]}
```

EQL properties, joins, and idents support parameterization. A parameter is express in an EDN list and params are defined in a map.

```clojure
; without params
[:foo]

; with params
[(:foo {:with "params"})]

; ast
{:type     :root
 :children [{:type         :prop
             :dispatch-key :foo
             :key          :foo
             :params       {:with "params"}
             :meta         {:line 1, :column 15}}]}

; join with params wrap the key with the list
[{(:join-key {:with "params"})
  [:sub-query]}]
; alternate syntax to add params on joins (wrap the entire map, AST result is the same)
[({:join-key
   [:sub-query]}
  {:with "params"})]
; ast
{:type     :root
 :children [{:type         :join
             :dispatch-key :join-key
             :key          :join-key
             :params       {:with "params"}
             :meta         {:line 1, :column 16}
             :query        [:sub-query]
             :children     [{:type         :prop
                             :dispatch-key :sub-query
                             :key          :sub-query}]}]}

; ident join with params
[{([:ident "value"] {:with "params"})
  [:sub-query]}]
; ast
{:type     :root
 :children [{:type         :join
             :dispatch-key :ident
             :key          [:ident "value"]
             :params       {:with "params"}
             :meta         {:line 1 :column 16}
             :query        [:sub-query]
             :children     [{:type         :prop
                             :dispatch-key :sub-query
                             :key          :sub-query}]}]}

```

EQL use `...` or a number as the join value subquery to support recursive queries.

```clojure
; the join value is ...
[:entry/name {:entry/folders ...}]
;ast
{:type :root,
 :children
 [{:type :prop, :dispatch-key :entry/name, :key :entry/name}
  {:type :join,
   :dispatch-key :entry/folders,
   :key :entry/folders,
   :query ...}]}

; you can bound the recursion limit using a number as a sub-query
[:entry/name {:entry/folders 3}]
;ast
{:type :root,
 :children
 [{:type :prop, :dispatch-key :entry/name, :key :entry/name}
  {:type :join,
   :dispatch-key :entry/folders,
   :key :entry/folders,
   :query 3}]}
```

Meta data can be stored on a query.

```clojure
(with-meta [] {:meta "data"})
; ast
{:type :root, :children [], :meta {:meta "data"}}
```

In EQL unions specify polymorphic requirements. Possible suq-queries are defined in a map as the join value. The key in each subquery is used to match the possible data.

```clojure
[{:chat/entries
  {:message/id [:message/id :message/text :chat.entry/timestamp]
   :audio/id   [:audio/id :audio/url :audio/duration :chat.entry/timestamp]
   :photo/id   [:photo/id :photo/url :photo/width :photo/height :chat.entry/timestamp]}}]
;ast
{:type :root
 :children
 [{:type         :join
   :dispatch-key :chat/entries
   :key          :chat/entries
   :query        {:message/id [:message/id :message/text :chat.entry/timestamp]
                  :audio/id   [:audio/id :audio/url :audio/duration :chat.entry/timestamp]
                  :photo/id   [:photo/id
                               :photo/url
                               :photo/width
                               :photo/height
                               :chat.entry/timestamp]}
   :children     [{:type :union
                   :query
                         {:message/id [:message/id :message/text :chat.entry/timestamp]
                          :audio/id   [:audio/id :audio/url :audio/duration :chat.entry/timestamp]
                          :photo/id   [:photo/id
                                       :photo/url
                                       :photo/width
                                       :photo/height
                                       :chat.entry/timestamp]}
                   :children
                         [{:type      :union-entry
                           :union-key :message/id
                           :query     [:message/id :message/text :chat.entry/timestamp]
                           :children  [{:type :prop, :dispatch-key :message/id, :key :message/id}
                                       {:type :prop, :dispatch-key :message/text, :key :message/text}
                                       {:type         :prop
                                        :dispatch-key :chat.entry/timestamp
                                        :key          :chat.entry/timestamp}]}
                          {:type      :union-entry
                           :union-key :audio/id
                           :query     [:audio/id :audio/url :audio/duration :chat.entry/timestamp]
                           :children  [{:type :prop, :dispatch-key :audio/id, :key :audio/id}
                                       {:type :prop, :dispatch-key :audio/url, :key :audio/url}
                                       {:type         :prop
                                        :dispatch-key :audio/duration
                                        :key          :audio/duration}
                                       {:type         :prop
                                        :dispatch-key :chat.entry/timestamp
                                        :key          :chat.entry/timestamp}]}
                          {:type      :union-entry
                           :union-key :photo/id
                           :query     [:photo/id
                                       :photo/url
                                       :photo/width
                                       :photo/height
                                       :chat.entry/timestamp]
                           :children  [{:type :prop, :dispatch-key :photo/id, :key :photo/id}
                                       {:type :prop, :dispatch-key :photo/url, :key :photo/url}
                                       {:type :prop, :dispatch-key :photo/width, :key :photo/width}
                                       {:type :prop, :dispatch-key :photo/height, :key :photo/height}
                                       {:type         :prop
                                        :dispatch-key :chat.entry/timestamp
                                        :key          :chat.entry/timestamp}]}]}]}]}
```

### 4.2 Mutations

Mutations in EQL represent operation calls that behave like event sourcing. They can be applied locally, across a network, onto an event bus, etc.

A mutation is represented by a list of two elements: a symbol representing an operation name, and a map representing the input data. For example:

```clojure
[(call.some/operation {:data "input"})]

; ast
{:type :root
 :children
 [{:dispatch-key call.some/operation
   :key          call.some/operation
   :params       {:data "input"}
   :meta         {:line 610, :column 17}
   :type         :call}]}
```

Both operations and query with parameters are represented by a list. The difference is that operations use a symbol where query with parameters uses keywords (properties, idents, and joins).

A mutation may have a return value described by a query. It is done by combining the syntax of a join with the syntax of a mutation.

```clojure
; mutation with a return value
[{(call.some/operation {:data "input"})
  [:response :key-a :key-b]}]
; ast
{:type :root
 :children
 [{:dispatch-key call.some/operation
   :key          call.some/operation
   :params       {:data "input"}
   :meta         {:line 612 :column 18}
   :type         :call
   :query        [:response :key-a :key-b]
   :children     [{:type :prop, :dispatch-key :response, :key :response}
                  {:type :prop, :dispatch-key :key-a, :key :key-a}
                  {:type :prop, :dispatch-key :key-b, :key :key-b}]}]}
```
