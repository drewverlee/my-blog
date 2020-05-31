{:title "Factory Method Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2017-5-08"
 :tags  ["Clojure" "Design Pattern" "Java" "Software" "Creational"]}

###Intent

The GOF pattern says the intent is:

> Define an interface for creating an object, but let subclasses decide which
> class to instantiate. Factory Method lets a class defer instantiation to
> subclasses.

* Here is a fairly human consumable [example](http://alvinalexander.com/java/java-factory-pattern-example).

* Here is an interesting  post on this pattern in [clojure](http://blog.wagjo.com/factory.html).
I say interesting because i can't tell if this is over engineered. 

The premises is you don't want your client to be found to a certain set of functionality.

Maybe it would be best if we looked at their example code?

```
Product* MyCreator::Create (Productld id) {
    if (id == YOURS)  return new MyProduct;
    if (id == MINE)   return new YourProduct;
        // N.B.: switched YOURS and MINE

    if (id == THEIRS) return new TheirProduct;

    return Creator::Create(id); // called if all others fail
}
```

Right, so its a switch statement.

### Clojure implementation 


I'm not sure what to do here. It seems like the problem is that you have some 
data and you want the code to handle it no matter what the type. Look at the
example code above, `id == Yours` is akin to a type check. In Clojure we have a method
for dispatching to functionality based on a function that will do the same

the difference is that instead of fetching the class, i'm going to just perform
the action we need to get the job done.


```klipse-cljs
(defmulti draw (fn [{:keys [shape]}] shape))

(defmethod draw :circle 
   [{:keys [shape]}] (str "drawing a  big " shape))

(draw {:shape :circle})
```

Things change and you want another shape?

```klipse-cljs

(defmethod draw :square
   [{:keys [shape]}] (str "drawing a  tiny " shape))

(draw {:shape :square})
```

However what type a  multimethod is dispatched is determined at compile time,
often we want to dispatch on some run time value. In which case we can juse use any number
of conditional control structures `if cond when`


Here is a quick example.

```klipse-cljs
(defn describe-number [n]
  (cond
    (odd? n) "odd"
    (zero? n) "zero"))
```

Then try it out:

```klipse-cljs
(describe-number 5)
```

I talk the various tradeoffs of chaining together functions 
on the Chain of Responsibility post.


### Summery

The factory pattern is about building up an object based on some input.
Objects are groups of functions. But in functional languages we don't group
together functions in a class, we call them as needed on the data. The factory method
seems to be a filler step between getting the data you need to the functions
you want to call on it. We eliminated the filler with a multimethod.












