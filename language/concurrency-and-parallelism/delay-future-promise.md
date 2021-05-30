# Delay, Future and Promise

The `realized?` returns true if a value has been produced for a promise, delay, future or lazy sequence.

## Delay

A `delay` takes a body of expressions that are executed the first time the `Delay` object is dereferenced by `deref` or `@`. The body is executed only once and the result is cached. It is often used to contain expensive computation result.

## Future
