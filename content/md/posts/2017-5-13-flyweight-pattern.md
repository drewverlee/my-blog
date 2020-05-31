{:title "Flyweight Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software" "Structural"]}


<img src="/img/sharing.jpg">

### Intent

The Gang of Four book says the intent is:

> Use sharing to support large numbers of fine-grained objects efficiently.

If you read more you go on to realize that the purpose is save memory by sharing state. Wikipedia is more direct:

> A flyweight is an object that minimizes memory usage by sharing as much data
> as possible with other similar objects; it is a way to use objects in large
> numbers when a simple repeated representation would use an unacceptable
> amount of memory

### In Clojure

Clojure collections (lists, maps, vectors, etc...) use structural sharing. Lets
just see it in action before we even do a light dive:

create a vector `a`:

```klipse-cljs

(def a [1 2])
```

Create another vector `b` using ag

```klipse-cljs
(def b (conj a 3))
```

Realize that `b`s 0th and 1st element are shared with a.

```klipse-cljs
(= a (subvec b 0 2))
```

This is possible because the collection is immutable. This [blog post](http://hypirion.com/musings/understanding-persistent-vector-pt-1) goes into
great detail about structural sharing in clojure, and i highly recommend it.

But this picture should give you an intuition. The high level idea is that the pink, orange and blue vector 
all share some elements and their is a tree structure that efficiently stores and remembers this. 

<img src="/img/7to10.png">


If you need to mutate that state so it changes between a and b
then take a look at [clojure state mechanisms](https://clojure.org/about/state).

### Summery

Clojure uses immutable data structure's which makes sharing a feature that you get out of the box. If you need mutability
then their is a rich core library. Finally, if you want really wanted was _caching_ on a function, then look at `memoize`.


