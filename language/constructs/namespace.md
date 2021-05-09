# Namespace

## Auto-resolved Keyword

`::` is used to auto-resolve keyword in the current namespace. If no qualifier is specified, it will auto-resolve to the current namespace. If the keyword is qualified, the namespace will be resolved using aliases in the current namespace. For example, if the current namespace has an aliase `example` for `x`, then `::x/foo` is resovled as `:example/foo`. `::` is not available in edn.

## Namespace Map

Namespace map is used to specify a default namespace context when keys or symbols in a map where they share a common namespace. The syntax is `#:ns{...}` that specifies a prfeix `ns`for all keys or symbols in a map.

```clojure
#:person{:first "Han"
         :last "Solo"
         :ship #:ship{:name "Millennium Falcon"
                      :model "YT-1300f light freighter"}}
; it is read as:
{:person/first "Han"
 :person/last "Solo"
 :person/ship {:ship/name "Millennium Falcon"
               :ship/model "YT-1300f light freighter"}}
```

Similarly, `#::{}` resolves the current namespace. `#::alias` works for namespace alias.
