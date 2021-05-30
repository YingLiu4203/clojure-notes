# Delay, Future and Promise

The `realized?` returns true if a value has been produced for a promise, delay, future or lazy sequence.

## Delay

A `delay` takes a body of expressions that are executed the first time the `Delay` object is dereferenced by `deref` or `@`. The body is executed only once and the result is cached. It is often used to contain expensive computation result.

## Future

A `future` takes a body of expressions that are executed immediately in a thread pool. It is an instance of `java.util.concurrent.Future`.

Like `delay`, the result is cached. Unlike `delay`, you can use a timeout when `deref` a future.

The `future-call` calls a no-arg function in a thread pool.

## Promise

A promise is a workflow variable that works like a one-time, single-value pipe. Initially `(promise)` is an empty container. You use `deliver p val` to set a value and `(deref p)` is a blocking call to get the value. Optionally, you can provide a timeout and a default value like `(deref p timeout default)`.

It is often used in `declarative concurrency`: a derivative result is calculated on demand as soon as its inputs are available.

If an async function takes a callback as its last argument, the following code implements a general-purpose sync function that takes an async function as its argument and returns a function that calls the async function, puts results into a promise and `deref` the promise.

```clojure
(defn sync-fn
  [async-fn]
  (fn [& args]
    (let [result (promise)]
      (apply async-fn (conj (vec args) #(deliver result %&)))
      @result)))
```
