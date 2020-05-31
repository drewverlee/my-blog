{:title "Chain of Responsibility"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software" "Behavioral"]}


<img src="/img/Relay.jpg">

###Goal


The Gof book describes the intent of the Chain of Responsibility as:

> Avoid coupling the sender of a request to its receiver by giving more than
> one object a chance to handle the request. Chain the receiving objects and
> pass the request along the chain until an object handles it.


The book has more to say, but the take away is that
the pattern reduces coupling between senders and receivers. This shounds 
a lot like the producer consumer pattern only without the requirement 
for concurrency. 

I would say we need to

* [  ] decouple sender from receiver
* [  ] default behavior for unhanded requests
* [  ] request should be handled by every handler that can handle it*



The last requirement 

### How we would express this in clojure

Assuming we just wanted the first three requirements and not the one to have 
multiple handlers then this is simple using cond:

```klipse-cljs
(defn describe-number [n]
  (cond
    (odd? n) "odd"
    (zero? n) "zero"))
```

No mystery how this works:

```klipse-cljs
(describe-number 5)
```

We have satisfied our first requirement, the client here isn't explicit calling
those inner expressions

* [x] decouple sender from receiver

The next thing to tackle is default behavior, we can just add something that always evals to true like a keyword.

```klipse-cljs
(defn describe-number [n]
  (cond
    (odd? n) "odd"
    (zero? n) "zero"
    :else "must be even then"))
```

```klipse-cljs
(describe-number 4)
```

In order to tackle

* [ ] request should be handled by every handler that can handle it*

We have to change the function were using a threading macro.
Threading functions have an `->` at the end and indicate that arguments pass through all
their arguments. 

```klipse-cljs
(defn describe-number [n]
  (cond-> []
    (odd? n) (conj "odd")
    (even? n) (conj "even")
    (zero? n) (conj "zero")
    (pos? n) (conj "positive")
    :else (conj "i always happen")))
```
But you will notice that :else is a bit of a lie, as it seemed to happen regardless:

```klipse-cljs
(describe-number 5)
```

So it would probably be best to just handle the default case outside of the macro

```klipse-cljs
(defn describe-number [n]
  (cond-> []
    (odd? n) (conj "odd")))


(defn default-handler [r]
 (if  (empty? r) ["even"] r))


(default-handler (describe-number 2))
```

Finally, if we want want to be able to change the handlers at run time we will need to
abandon `cond` and use [reduce](https://clojuredocs.org/clojure.core/reduce). Reduce is a workhorse 
of clojure. 


```klipse-cljs
;; we define an extra function just to show were not limited to core clojure functions...
(defn big? [n] (> n 10))

(defn describe-number [n clauses]
  (reduce
    (fn [{:keys [r result]} {:keys [pred claus]}]
      (if (pred r)
        {:r r :result (conj result (claus r))}
        {:r r :result result}))
    {:r n :result []}
    clauses))
```

We have factored out the clauses from the code that runs them.

```klipse-cljs
(describe-number
  11
  [{:pred odd? :claus (fn [r] "odd")}
   {:pred even? :claus (fn [r] "even")}
   {:pred big? :claus (fn [r] "big")}])
```


### Summery

Chain of responsibility is about decoupling senders and receivers. We showed how `cond` allowed
us to have multiple handlers, but if we wanted the handlers to be determined at run time, we 
had to abandon it. This is because cond is a macro and so the arguments i hand it must adhere to a structure at compile time,
and that structure doesn't allow for arbitrary clauses.

Luckily were able to apply reduce without much head scratching to solve our problem. It's also easy to fit
more complex situations. For instance, it would be very powerful to only allow certain transactions of handlers
 the ["debit-account"] can only happen if the event is also handlable by the ["transfer money"] handler. We could do this by
 not actual firing the side effects until the end of the handler chain then validating the effects block. 

 However the big conclusion is that...

 *Chain of responsibility simplified was made possible by having fully supported first class functions*











