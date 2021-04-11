# Transducer

- [Tranducers Doc](https://clojure.org/reference/transducers)
- [Mental Model](https://functional.works-hub.com/learn/a-mental-model-for-thinking-about-clojures-transducers-6baba)
- [Transducer by Rich](https://youtu.be/6mTbuzafcII)
- [Inside Transducer Video](https://youtu.be/4KqUvG8HPYo)

## Motivation

A reducing function is any function that takes an accumulator and a new value and generates a new accumulator: `(accumulated-result, input) -> accumulated-result`. A reducing function implementation has the following steps:

- processing an input with the current accumulated result
- creating and returning an accumalted result

A fact (may or may not be a problem) of a reducing function is that it couples/fixes sequendce builder: the operation that produces the new accumulated result. That operation could be `conj` to create a new sequnence or a bineary operator such as `+` to create a scalar value. Of course, the initial value will be different for different operations. For `conj`, the initial value could be a `[]`; for `+`, it could be `0`.

If we add a constraint and a parameter to a reducing function, we can achieve a goal of composing multiple reducing functions to process each element and generate an accumulated result in a single operation.

- Constraint: all recuders to be composed take the same accumulate type. This reducing function is actually a transformer.
- reducing function parameter: a sequence builder to created a result, after applied composed transformers. This is the actual reducing operation.

## Goal

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

## An Example

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
(defn template-transducer [xf]
    (fn
      ([] (xf)) ;; Setup
      ([result input] (xf result input)) ;; Process, logic here
      ([result] (xf result)))) ;; result

(defn mapping
  [f]
  (fn [xf]
    (fn
      ([] (xf))
      ([acc] (xf acc))
      ([acc v]
       (xf acc (f v))))))


(defn filtering
  [f]
  (fn [xf]
    (fn
      ([] (xf))
      ([acc] (xf acc))
      ([acc v]
       (if (f v)
         (xf acc v)
         acc)))))
```

Core sequence functions' collectionless arity now returns a transducer: `map`, `mapcat`, `filter`, `remove`, `take`, `take-while`, `drop`, `drop-while`, `take-nth`, `replace`, `partition-by`, `partition-all`, `keep`, `keep-indexed`, `cat`, `dedupe`, `random-sample`...

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

## Summary

- All transducers and the reducing function have the compatible input and output types.
- The reducing function provides the initial value and the final result from the final accumulated values.
- Each transducer produces a new result value from current accumulated value and a input. the new result value is used as an input accumulated value for the next transducer.
- Arity-0 transducer is used to provide initial value from the reduceer when the transducer is the last composed transducer (bottom transducer).
- Arity-1 transducer is used for completion to provide final result from the reducing function. It is called in each transducer when process is done.
- Arity-2 transducer is used to process each input value. It must preseve the accumulated values and optionally process new input.
