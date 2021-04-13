# Transducer

- [Tranducers Doc](https://clojure.org/reference/transducers)
- [Mental Model](https://functional.works-hub.com/learn/a-mental-model-for-thinking-about-clojures-transducers-6baba)
- [Transducer by Rich](https://youtu.be/6mTbuzafcII): Usage and insisghts by Rich.
- [Inside Transducer Video](https://youtu.be/4KqUvG8HPYo): Rich's another talk.
- [Clojure Tranducers](https://youtu.be/-hwWZkqQO9w): a good explanation.

Some history blogs:

- [Tranducers are Coming](https://cognitect.com/blog/2014/8/6/transducers-are-coming)
- [Anatomy of a Reducer](https://clojure.org/news/2012/05/15/anatomy-of-reducer)
- [Reducers](https://clojure.org/news/2012/05/08/reducers)

## Definitions

- `Reducing Function (rf)`: the first parameter of `reduce` funciton. A reducing function is any function that takes an accumulator and a new value and generates a new accumulator: `accumulator * input -> accumulator`. In addition, a reducing function maybe called with no args and return an identity value for its operatoin. `() -> identity-value`.
- `Transducer`: is a reducing function transformer with a signature of `reducing-function -> reducing-function`, or the expanded form: `(accumulator * input -> accumulator) -> (accumulator * input -> accumulator)`. It is composable.
- `Step`: A arity-2 function defined in a transducer to process input.
- `Transducible Process`: A transducible process is defined as a succession of steps where each step ingests an input. The source of the inputs is specific to each process and must choose what to do with the outputs produced by each step.

A transduceer don't care what the reducing funciton is, the accumlator representation, and the source of inputs. Most sequnce function such as `map`, `filter` can create a transducer by applying to a function like `(map f)`.

## Motivation

A reducing function implementation has the following steps:

- processing an input with the current accumulated result
- creating and returning an accumalted result

In early days, without `reduce`, a typical implementation of a reducing function has a signature like `fn * sequable -> sequable`. It complects many `how`:

- recursion mechanism
- sequential order navigation
- laziness (mostly lazy)
- collection builder/representation

You want to decomplect the four things from reducing.

## Goals

Extract the essence of map, filter et al away from the functions that transform sequences/collections, thus they can be used elswhere. Recasting them as process transformations.

The kind of processes are:

- can be modeled as a succession of steps.
- where each step ingests an input
- seeded left reduce is the generalization where building a collection is just one instance

Why transducer:

- reduce: lead back a sequence
- ingest: carry into one thing
- transduce: lead across a set of functions
- on the way back/in, will carry inputs across a series of transformations.

Traditional map, filter, mapcat take a sequence and return sequence. Every new collection/process defines its own version of map, filter, mapcat et al. Composed algorithms are specific and inefficient.

Transducers know nothing about the step function (the process context) they modify. Reducing function is fully encapsulated. It may call step 0, 1, or more times. It must pass previous result as the result parameter (the first parameter) of the step function. It can transform the input argument, the 2nd argument.

Transducers can be composed to a transducer that can be composed again and again. Ransducers modify a process by `transforming its reducing function`. The transducer can be used in different context such as putting into a collection, a lazy sequence, a `transduce` call, a channel, an `observable`.

A transducer takes a function, wraps it and returns a new step function. Composing the transformers has an order of right to left but it yields input transformations that run left to right. Transducers are fast because it is just a stack of function calls, no laziness, no interim collections and extra boxes.

`into`, `sequence`, `transduce`, `chan` etc accpt transducers to transform their (internal, encapsulated) reducing function (step) and do their job with the transformed reducing funciton. They are transducible processes. For example, `into` use `conj` as reducing function to put inpt into a collection. `chan` adds an input into a buffer.

Processes must support `reduced` in case of early termination. It stops supplying input to the step function. Some transducers require state. For example, `take`, `partition-*`.

## A Mental Model

The `reduce` function introduces a generic way to apply a reducing function to a collection. It abstracts the first three items (mechanism, order and laziness) and uses a reducing function to process input and build a new sequence. For example:

```clojure
(defn reduce-map [f coll]
  (reduce (fn [acc v]
            (conj acc (f v)))
          []
          coll))
```

A collection or anything that you can `reduce` is called `reducible`. A reducing function in `core.reducer` has a signature like `fn * reducible -> reducible`.

This reducing function couples/fixes sequence builder: the operation that produces the new accumulated result. That operation could be `conj` to create a new sequnence or a bineary operator such as `+` to create a scalar value. Of course, the initial value will be different for different operations. For `conj`, the initial value could be a `[]`; for `+`, it could be `0`. The initial value for `reduce` depends on the sequnce builing operation.

The sequence builder `conj` and `+` are also reducing functions. Extracting this reducing function by taking it as a parameter of a reducer, the reducer has a signature of `reducing-function -> reducing-function`.

```clojure
;; mental model of the map reducer
;; use rf for reducing-function, f is from a transducer
(fn [rf]
  (fn [acc v]
    (rf acc (f v))))
```

The reducer is decompleted from both representation and order. With reducer, you can define a transducer as `fn -> reducer`. A transducer function takes a transformation function parameter and returns a reducer. Here the `fn` is the transformation part of the `transducer`. For example:

```clojure
(defn map-transducer [f]
  (fn [rf] ;; the map reducer
    (fn [acc v]
      (rf acc (f v)))))
```

The reducer implementation is transducer-specific. A reducer gets the transformation function `f` from its transducer and gets the reducing function from its caller. The reducer returns a reducing function. In its reducing funciton implementation, the input reducing function is treated as the next step that is called after the processing of the new input by applying `f` to the input.

It is an important detail that the reducer calls the reducing function after applying `f` to its input. It is the reason that a composed transducer works in the reversed order of `comp` function.

If you expand a tranducer, you get something like `[f, rf, acc, v] -> result`. The first parameter `f` is provided at call time to customize the transformation, the reducing function is provided by a process such as `transduce`/`chan` that calls `reduce` eventually. The `reduce` applies `transducer(f)(rf)` to a reducible. Mentally you can think a transducer having a signature of `fn * reducible -> reducible`.

## What You Get

The transducer decompletes the following hows:

- How: mechanism - functional transformation of reducing function
- How: order - doesn’t know
- How: laziness - doesn’t know
- How: representation - doesn’t build anything

The call of `(transducer f)` returns a `reducer` that is a `transformation of reducing function`. It works for many sequence functions such as Core sequence functions' collectionless arity now returns a transducer: `map`, `mapcat`, `filter`, `remove`, `take`, `take-while`, `drop`, `drop-while`, `take-nth`, `replace`, `partition-by`, `partition-all`, `keep`, `keep-indexed`, `cat`, `dedupe`, `random-sample` etc. It is general that the tansformation could be 1:1, contractive or expansive.

The `reduce` process is based on `foldl` that uses an accumulator and is eager and serail. Using different processes such as `fold` brings parallel processing by using JVM's fork-join thread pool.

It can be composed and can be used in different processes.

- reduce with a transformation (no laziness, just a loop): `(transduce xform + 0 data)`.
- construct a new collection: `into [] xf data)`.
- lazily transform the data (one lazy sequence, not three as with composed sequence functions): `(sequence xform data)`.
- build one collection from a transformation of another, again no laziness: `(into [] xform data)`.
- create a recipe for a transformation, which can be subsequently sequenced, iterated or reduced: `(eduction xform data)`.
- use the same transducer to transform everything that goes through a channel: `(chan 1 xform)`.

## Examples

```clojure
(defn trans-mapping [f]
  (fn [step]
    (fn [acc v]
      (step acc (f v)))))

(defn trans-filtering [f]
  (fn [step]
    (fn [acc v]
      (if (f v)
        (step acc v)
        acc))))

;; apply step in each element in a collection
;; this is a transducer
(def cat
  (fn [step]
    (fn [acc v] (reduce step acc v))))

(def mapcatting [f]
  (comp (map f) cat))

(def rfn (comp (trans-mapping inc)
               (trans-filtering even?)))

((rfn +) 42 5)

; expanded as the following
(((trans-mapping inc)
  ((trans-filtering even?) +)) 42 5)

; expanded one more level
((fn [acc v]
   (if (even? v)
     (+ acc v)
     acc)) 42 (inc 5))
```

There are some important details in the above code:

- The `comp` applies function arguments from right to left. The function argument such as `(trans-mapping inc)` and `(trans-filtering even?)` take a sequence builder function `xf` as their argument, here it is `+`. When applied the composed function result to data, the `f`s are applied from left to right. `f` is executed before `xf` and the right function argument becomes the `xf` of the left function argument whose `f` runs first. Here the sequence is `inc` followed by `even?`.
- As shown in the expanded syntax, the `(trans-filtering even?) +)` is passed as the `xf` to `(trans-mapping inc)` because after applying `f` and `xf`, both `trans-filter` and `trans-map` return the same `fn [acc v]`. The `xf` takes the same input `[acc v]` and is shared by both composed functions.

## Reducing Function

The sequence builder `xf` is parameterized. Because `xf` is closely coupled with the initial value of `acc` and the final result, it might be a good idea to let `xf` to have one arity that initializes the value and one arity for the result value. A reducing function function is a function has three arities: 0, 1 and 2.

```clojure
;; a reducing function is a multi-arity function
(defn string-rf
  ;; this is to build a new string-builder
  ([] (StringBuilder.))
  ;; this is to convert the return value
  ([^StringBuilder s] (.toString s))
  ;; this is the construction
  ([^StringBuilder acc ^Character c]
   (.append acc c)))

;; template
(defn reducing-function
  ;; this is to build a new string-builder
  ([] init_value))
  ;; this is to convert the return value
  ([result] (get_result(result)))
  ;; this is the construction
  ([result input] (accumulate(result, input))))
```

## Transducers Take and Return Reducing Functions

A transducer is a function that takes a reducing function and returns a new reducing funciton.

```clojure
;; template
(defn template-transducer [rf]
    (fn
      ([] (rf)) ;; Setup
      ([result input] (rf result input)) ;; Process, logic here
      ([result] (rf result)))) ;; result, may clean/up or transform result

(defn mapping
  [f]
  (fn [rf]
    (fn
      ([] (rf))
      ([acc] (rf acc))
      ([acc v] (rf acc (f v))))))


(defn filtering
  [f]
  (fn [rf]
    (fn
      ([] (rf))
      ([acc] (rf acc))
      ([acc v]
       (if (f v)
         (rf acc v)
         acc)))))
```

The multi-arity functions:

- Arity-0 (init) is used to provide initial value from the reducer when the transducer is the last composed transducer (bottom transducer).
- Arity-1 (completion) is used for completion to provide final result from the reducing function.
- Arity-2 (step) is used to process each input value. It must preseve the accumulated values and optionally process new input.

Clojure has a mechanism for specifying early termination of a reduce:

- `reduced` - takes a value and returns a reduced value indicating reduction should stop.
- `reduced?` - returns true if the value was created with reduced.
- `deref` or `@` can be used to retrieve the value inside a reduced.

Use `volatile!` to store state in the closure of a transducer. It has better performance than `atom`.

## `transduce`

```clojure
(defn transduce
  ([xform f coll] (transduce xform f (f) coll))
  ([xform f init coll]
   (let [xf (xform f) ;; transformation of reducing step
         ret (reduce coll xf init)]
     ;; completion
     (xf ret))))
```

An exmaple:

```clojure
;; and now the xform can look like
(def xform
  (comp (mapping int)
        (mapping inc)
        (filtering even?)
        (mapping char)))

(transduce xform string-rf "Hello World")
```

## Transducible Processes

A transducible process must follow the following rules:

- If a step function returns a reduced value, the transducible process must not supply any more inputs to the step function. The reduced value must be unwrapped with deref before completion.
- A completing process must call the completion operation on the final accumulated value exactly once.
- A transducing process must encapsulate references to the function returned by invoking a transducer - these may be stateful and unsafe for use across threads.
