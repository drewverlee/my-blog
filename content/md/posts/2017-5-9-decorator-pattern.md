{:title "Decorator Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software" "Structural"]}

<img src="/img/sandwhich.jpg">

### Intent

The GOF says the intent is:

> Attach additional responsibilities to an object dynamically. Decorators
> provide a flexible alternative to subclassing for extending functionality.

The idea is to give the client the ability to happens before and after some functionality.
Thats actual too vague to point us in any one direction.


```klipse-cljs
; create some simple functions
(defn add-one [n] (+ n 1))
(defn add-two [n] (+ n 2))
(defn times-ten [n] (* n 10))

; a function that lets us apply them in any order we want...
(defn do-something [n & fs]
  (map (apply comp fs) n))


(do-something [1 2 3] add-one times-ten)
```

Or use a different set of transformations:

```klipse-cljs
(do-something [1 2 3] add-one add-two)
```

Or maybe all you want to do is just wrap a function in another function


```klipse-cljs
(defn g [arg f]
  (+ 1 (f arg)))

(g 2 #(* 10 %))
```

Their are more powerful ways to control the transformations though. Consider
building a pipeline of transformations where the next function in the chain actual
controls previous functions from even getting called.

Dont get to hung up on the code at this point.

```klipse-cljs
(defn middleware-factory 
  [next-handler postivity]
  (fn [greeting]
    (if (= "you suck" greeting)
      "I dont have to take that kind of abuse"
      (let [y (next-handler greeting)]
        (if (clojure.string/includes? y postivity)
          (str  greeting " to you as well")
          (str "you can do better then... " y))))))


(def greet (middleware-factory (fn [greeting] (str greeting "!"))  "good"))
```

The thing to note is the behavior we achieve. Their are a couple different paths.

We don't use the output of the next function (adding a "!" the greeting):

```klipse-cljs
(greet "good day")
```

not only don't we use the output of the next function, we dont even *call* it based on the arguments.

```klipse-cljs
(greet "you suck")
```

We both call the next function and call its output:

```klipse-cljs
(greet "meh")
```

This is maybe the first functional pattern i have touched on in this series. Does that mean clojure is missing some feature?
I dont think so, i think this is just sufficiently complex that it warrants a name and that name is `middleware factory`
you can read more about how its used as part of a build tool called [BOOT](http://www.braveclojure.com/appendix-b/).

But the fun doesn't stop here. What if you want to control the entire pipeline of functions at any given point? [Pedestal](https://www.youtube.com/watch?v=_Cf-STRvFy8&t=1091s)
has whats called an interceptor chain, which does just that.

### summery

First class functions mean wrapping/decorating other functions is straight forward. It can be done by passing the functions into each other or chaining them together.
