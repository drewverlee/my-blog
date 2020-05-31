{:title "Abstract Factory Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2017-4-12"
 :tags  ["Clojure" "Design Pattern" "Java" "Software"]}

<img src="/img/factory.jpg"

###Intent

Design Patterns:  Elements of Reusable Object-Oriented Software says the intent of the
Abstract Factory Principle is:

> Provide an interface for creating families of related or dependent objects
> without specifying their concrete classes.

For a visual reference go [here](https://dzone.com/articles/design-patterns-abstract-factory).
For an example in clojure that is more true to the original pattern check [this out](https://gist.github.com/didibus/7a7e2f9c9740d10159bdd3c1651fc43f).

###Implementation in clojure

An object should just be a bag of functions. So we we want to build up a bag of functions in clojure
we have several options. We can  pass the functions into another function. This is one way to create a family of related
functions.

```klipse-cljs
(defn f [x] (+ 1 x))
(defn g [x] (* 3 x))
(defn foo [f g x] (g (f x)))

(foo f g 3)
```

We could compose the functions together into one function using [comp](https://clojuredocs.org/clojure.core/comp):

```klipse-cljs
(def z (comp g f))
(z 3)
```

Or we could build up the function by supplying some if its arguments using [partial](https://clojuredocs.org/clojure.core/partial):

```klipse-cljs
(def fooy (partial foo f g))

(fooy 3)
```

Or we can create a bag of functions very much like an object:

```klipse-cljs
(def obj {:f f :g g})

((obj :f) 2)
```

But the Abstract Factory Method is about decoupling the client from the bag of functions/object
its using. This way we can add new bags of functions/objects without breaking the client. We can
also possible let the choice of functions happen at run time. 

Allowing a function to behave different depending on the type is polymorphism. 
In Clojure we have two tools that achieve this, [mulimethods and protocals](https://8thlight.com/blog/myles-megyesi/2012/04/26/polymorphism-in-clojure.html).
Were going to employ multimethods to build a car factory. Were not going to use the word factory in any function
though, as it doesn't convey anything interesting here.  First we create some functions that are used to build the car. These functions dont do anything, but you can imagine whey they would do.


```klipse-cljs
(defn add-wheels [])
(defn build-model-one-body [])
(defn good-quality [] "good")
```

Were going to bundle those together into a map so we can pass them together and so the functions have alias/keys.
Later will pass those to the function that builds the car so it can take advantage of them:

```klipse-cljs
(def fancy-model-one {:build-body build-model-one-body :add-wheels add-wheels :quality good-quality})
```


Now we add the multimethod upon-which our polymorphic dispatch rests. This will allow us to build different
models of cars.  If it helps you can think of the multimethod like the interface.

```klipse-cljs
(defmulti build-car
  (fn ([{:keys [model]}] model)))
```

And a method we can dispatch to, this is like the concrete implementation. 

```klipse-cljs
(defmethod build-car "v1"
  [{:keys [wheels body fns] :as order}]
  (str "building a " ((fns :quality)) " model 1 car"))
```

Lets build a car to make sure everything works the way we expect. Notice we pass
in part of the functions to build the car. This is a design choice. We show some other
ways we build up our result later.

```klipse-cljs
(build-car {:model "v1" :wheels "g7s" :body "gold" :fns fancy-model-one})
```

Great! But we just got word from the higher ups that we need to make some cut backs
to improve revenue. so were offering the same model car but only its built by interns, 
were going to call this a `bad-quality` car. Half the price 1/4 the car!

The `quality` function determines the quality, so its what we need to change. How can we go about doing this?
Play around below and see what you come up with.


```klipse-cljs
;; try to make the change yourself
```

Because we injected the good-quality function, into the build function we can just create a new bag of options with
a bad-quality function swapped for the good one. We can even give this a name if it helps:


```klipse-cljs
(defn bad-quality [] "shitty")

(def cheap-model-one {:build-body build-model-one-body :add-wheels add-wheels :quality bad-quality})
```

Now we can build or bad car as easily as calling the build function with our new collection of functions:

```klipse-cljs
(build-car {:model "v1" :wheels "g7s" :body "gold" :fns cheap-model-one})
```

We said earlier that the Abstract Factory Pattern was about letting the client choose 
different functions depending on the data without having to know about concrete implementation
of those functions. We already satisfied this with our multimethod, but we didn't show it happening. 
Lets fix that by adding a requirement for a new model "v2"


```klipse-cljs
(defmethod build-car "v2"
  [{:keys [wheels body fns] :as order}]
  (str "building a " ((:quality fns)) " model 2 car its way better then a model 1"))
```

Now we need a new bag of functions to build our model v2. Luckly we can re-use some of the ones we already built!

```klipse-cljs
(defn build-model-two-body [])
(def fancy-model-two {:build-body build-model-two-body :add-wheels add-wheels :quality good-quality})

(build-car {:model "v2" :wheels "g7s" :body "gold" :fns fancy-model-two})
```

Finally, let's say we don't want to let the client worry about the build fns `build-model-x`. We could handle
this any number of ways. For instance, we just dispatch to a function that already has the functions contained inside it.
<i>We append an x on the function name just so our namespace doesn't get polluted.</i>

```klipse-cljs
(defmulti build-car-x (fn [{:keys [model]}] model))

(defmethod build-car-x "v2"
  [{:keys [wheels body] :as order}]
  (let [fns fancy-model-one]
    (str "building a " ((:quality fns)) " model 2x car its way better then a model 1")))

(build-car-x {:model "v2" :wheels "g7s" :body "gold"})
```

And we have done everything the Abstract pattern set out to achieve. Or so it seems,
lets check what the GoF pattern lists as Consequences to be sure.

###Consequences:

> It isolates concrete classes. The Abstract Factory pattern helps you control
> the classes of objects that an application creates. Because a factory
> encapsulates the responsibility and the process of creating product objects,
> it isolates clients from implementation classes. Clients manipulate instances
> through their abstract interfaces. Product class names are isolated in the
> implementation of the concrete factory; they do not appear in client code.

We isolated with our multimethod. A multimethod can dispatch via a function instead of just data, so its more flexible. If 
anything is "better" in software its the right kind of flexability, which this is.


> It makes exchanging product families easy. The class of a concrete factory
> appears only once in an application—that is, where it’s instantiated. This
> makes it easy to change the concrete factory an application uses. It can use
> different product configurations simply by changing the concrete factory.
> Because an abstract factory creates a complete family of products, the whole
> product family changes at once. In our user interface example, we can switch
> from Motif widgets to Presentation Manager widgets simply by switching the
> corresponding factory objects and recreating the interface.

We switch to which version of multimethod we dispatch to by changing the data we pass to the multimethod (v1 -> v2).
I would say this is more powerful as now were capturing that information in data, rather then code. For example,
we could potential intercept the call and change it.

> It promotes consistency among products. When product objects in a family are
> designed to work together, it’s important that an application use objects
> from only one family at a time. Abstract Factory makes this easy to enforce.

Our interface hides, but doesn't really enforce unless we couple the functions to the 
disported function. Was the keys of our functions e.g `cheap-model-one`  enough 
to enforce anything? I would say no, if anything we congregated ourselves on open ended it was.
I think we could gain more enforcement via protocals, required parameters and an [information model](https://youtu.be/kP8wImz-x4w?t=14m52s).
But honestly, without a specific use case, i think its impossible to implement enforcement. For example,
if there is some state we need to guard, then that would be better left to [Validators](https://clojuredocs.org/clojure.core/set-validator!)
which won't allow the inconstant state to exist explicitly. 

> Supporting new kinds of products is difficult. Extending abstract factories
> to produce new kinds of Products isn’t easy. That’s because the
> AbstractFactory interface fixes the set of products that can be created.
> Supporting new kinds of products requires extending the factory interface,
> which involves changing the AbstractFactory class and all of its subclasses.
> We discuss one solution to this problem in the Implementation section.

Here is the big win for Clojure. By leveraging first class functions we were 
able to pass -in- functionality and so easily produce  new kinds of cars. Java
supports first class functions now, so maybe that would change how the authors
would have implemented this and many other patterns.

##Summery

The Abstract Factory Pattern is about polymorphic dispatch to a group of related functions. It's
also about enforcing how those related functions behave though. 
In Clojure have a number of ways we can group functions together depending on the use case. Without a specific
example its hard to say which would be best. We can enforce behavior but I would need a specific example
to know how best to do that, so we ignored it and assumed a more open system was better.

Some [would argue](http://mishadoff.com/blog/clojure-design-patterns/#abstract_factory) we could have satisfied this
pattern with just `partial`. However, this would be misleading. I think the factory pattern is
applicable in clojure, but it's either trivialized by use of polymorphic dispatch and dependency injection
or realized in full but only rarely, like when defining a [new data structure](http://blog.wagjo.com/factory.html).

As we have seen in previous patterns, the principles the pattern promotes remain intact by the implementation is supplied
by Clojures approach.



