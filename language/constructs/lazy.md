# Lazy

`dorun`, `doall`, and `doseq` are all for forcing lazy sequences, presumably to get side effects.

- `dorun` - don't hold whole seq in memory while forcing, return `nil`.
- `doall` - hold whole seq in memory while forcing (i.e. all of it) and return the seq
- `doseq` - same as dorun, but gives you chance to do something with each element as it's forced; returns `nil`.
- `for` is different in that it's a list comprehension, and isn't related to forcing effects. doseq and for have the same binding syntax, which may be a source of confusion, but doseq always returns `nil`, and for returns a lazy seq.

```clojure
;; forces through the sequence, returns nil.
(defn dorun [coll]
  (when (seq coll) (recur (next coll))))

;; forces but return the lazy sequence
(defn doall [coll] (dorun coll) coll)

;; for explaination purpose only, not real
(defmacro doseq [seq-exprs & body]
  `(dorun (for ~seq-exprs ~@body)))
```
