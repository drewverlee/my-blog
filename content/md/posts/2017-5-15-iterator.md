{:title "Interator Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software" "Behavrioral"]}


### Intent

>Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

This is straight forward.

### In clojure

In clojure collections:

* lists
* sets
* hashmaps
* vectors

Implement the [sequences](https://clojure.org/reference/sequences) interface.

> Seqs differ from iterators in that they are persistent and immutable, not stateful cursors into a collection.

So you can call first on any collection..

```klipse-cljs
(first [1 2])
(first #{1 2})
(first {:a 1 :b 2})
(first '(1 2))
```

or rest:

```klipse-cljs
(rest [1 2])
(rest {:a 1 :b 2})
(rest '(1 2))
(rest #{1 2})
```

Notice that what we get back from  `(rest #{1 2})` is actual a seq, not a vector `#{}`.


```klipse-cljs
(seq? (rest #{1 2}))
```

### Summery

Sequences are a deep fundamental topic in Clojure, i suggesting reading more about [here](http://insideclojure.org/2015/01/02/sequences/)
In clojure we _rarely_ define new data structures, but if you did, then it would probably be worth considering haveing 
them implement the Sequence interface. 
