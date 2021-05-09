# Excpetions

Exceptions are simple in concept: they can olny be thrown, caught, and logged. But there are many special cases in practice.

Resources:

- [Exceptions in Clojure](https://grishaev.me/en/clj-book-exceptions/)

## Basics

Clojure uses JVM excpetion handling: an exception is an object that you can `throw`. Exceptions can be chained using `cause` argument to point to nested exception. An application should handle all exception eventually to not let exceptions leaked to end users.

Use `try`, `catch` and `finally` to handle exceptions. The catch form takes a class and an arbitrary symbol. An exception will be bound to that symbol if control is taken to this branch. Use `ex-message e` to get exception message. You can have multiple `catch` forms to catch multiple exception types. `Throwable` matches all exceptions.

Use `format` to format message because `str` display nothing for `nil` value.

Clojure uses the result of a `catch` branch but ignores the return of `finally`.

## Use Exception

Clojure provides `ExceptionInfo` exception class that can store exception data. Use `ex-info` to create an object: `(throw (ex-info message map-data cause))`. The map-data is the context that cannot be `nil`. `ex-data` function gets this context.

A common case is validating data shapes using spec.

When `println` an exception, the root exception is printed at the beginning. The `:trace` has the stack trace.

The `clojure.stacktrace` package has several functions for printing exceptions. `print-throwable` shows a message and a context. `print-stack-trace` and `print-cause-trace` functions print trace with specified depth.

When there are many possible errors, use `(when-not (check1) error1)` to check each condition.

Context manager's name begins with the prefix `with-`. Clojure provides `with-open` to close resources.

## Logging

In a distributed system, message collectdion should be centralized. Logging gets messages and sends them to proper channels.

To use logback as the backend, add dependency `[ch.qos.logback/logback-classic "1.2.3"]` and put XL setting file in `resources` folder.

## Common Patterns

- Return a tuple of `[ture result]` or `[false exception]`.
- Callback takes `[error result]` pair.
- Retry with delay
- `recur` cannot be inside `try`, use a return tuple.
