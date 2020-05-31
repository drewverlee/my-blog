{:title "Adapter Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2017-5-08"
 :tags  ["Clojure" "Design Pattern" "Java" "Software"]}


###goal


The GOF book states the goal of the Adapter pattern is:

> Convert the interface of a class into another interface clients expect.
> Adapter lets classes work together that couldn’t otherwise because of
> incompatible interfaces.

Its applicable when:

> you want to use an existing class, and its interface does not match the one you need.

> you want to create a reusable class that cooperates with unrelated or unforeseen classes, that is, classes that don’t necessarily have compatible interfaces.

### Implementation in clojure

So the goal is to extend an existing interface, one we don't own the source code to.
In python or ruby we would monkey patch. In clojure their is no monkeying around,
we can just extend the class.

Say we have some reversible things and we want to make our tie reversible. The
logic here is irrelevant, as i dont know what a reversed tie would be. The point is that
we can extend the protocal (some set of functions) to operate on both a class we own (ReversibleTie)
and one we dont (js/String):

```klipse-cljs
(defrecord ReversibleTie [a b])

(extend-protocol IReversible
              ReversibleTie
              (^clj -rseq [tie] (->ReversibleTie (:b tie) (:a tie)))
              js/String
              (^clj -rseq [s] (str "We Welcome all your " s)))
```
Thats complaining probably because were extending an existing type with an existing method, which isn't a great idea...

So we make a tie, by instantiating a record, which at this stage you can think of
like a ordered map with required keys:

```klipse-cljs
(map->ReversibleTie {:a "a" :b "b"})
```

Now reverse it:

```klipse-cljs
(reverse
 (map->ReversibleTie {:a "a" :b "b"}))
```

And notice we can also "reverse" a String:

```klipse-cljs
(reverse "WhiffleCakes")
```

So we have effectively used an existing implementation and so satisfied the adapter pattern


### Summery

Clojure gives us a way to extend existing types and protocols, no need for any workarounds or patterns.

