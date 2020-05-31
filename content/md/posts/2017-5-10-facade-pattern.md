{:title "Facade Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software" "Structural"]}


### Intent

The Gang of Four book says the intent is:

>  Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.

The facade pattern takes some set of functionality and ties it together to create another function that a caller can use without having to
know the details.

### In clojure

In clojure if we want to tie together two functions you just create create another one that ties them together:

Lets say we have some functions for making pizzas

```klipse-cljs
(defn add-crust  [pizza] (conj pizza "crust"))
(defn add-thin-crust  [pizza] (conj pizza "thin crust"))
(defn add-cheese  [pizza] (conj pizza "cheese"))
(defn add-goat-cheese  [pizza] (conj pizza "goat cheese"))
(defn add-pepperoni  [pizza] (conj pizza "pepperoni"))
(defn add-ham  [pizza] (conj pizza "ham"))
```

But we have a new client that just wants his regular pizza and damnit he doesn't want to have to think about
some hippister goat cheese option. So we make a facade `make-regular`

```klipse-cljs
(defn make-regular [pizza]
 (-> pizza
     add-crust
     add-cheese
     add-pepperoni))

(make-regular [])
```

We don't expose how to make a regular pizza to our client, he gets his pizza, were all happy.

### summery

We can use first class functions to easily create a facade.
