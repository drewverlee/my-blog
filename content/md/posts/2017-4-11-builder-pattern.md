{:title "Builder Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2017-4-11"
 :tags  ["Clojure" "Design Pattern" "Java" "Software"]}

<img src="/img/builder.jpg">

###Goal

The GOF books states that the builder pattern is to:

> Separate the construction of a complex object from its representation so that the same construction process can create different representations.

However we get much more information out of what the GOF book states as consequences of using the Builder Pattern:

1. It lets you vary a product's internal representation by hiding the internal
  representation from the client. To produce a new product define simply definea
  new builder

2. It isolates code for construction and representation. A client doesn't know
  anything about the object that built the product or the product itself.

3. It gives you finer control over the construction process. You can take
  actions between steps to build up the result.

Lets make sure we understand the value of each of these:

1. Hiding the products internal representation from the client is important, because it means the products internal
representation can change without the client knowing.
2. Hiding how the builder works, is valuable because it means the builder can change without the client knowing.
3. Control over the construction process is valuable because now client(s) have control over building the data. 


###Implementation in Clojure:

Lets give ourselves some concrete tasks so we know were done. The third one is just an example as its hard to nail down concretely:

So the implementation needs to:
* <img src="/img/checkbox.png" height="15" width="15"> Hide the products internal representation from the client.
* <img src="/img/checkbox.png" height="15" width="15"> Hide how the builder works from the client. 
* <img src="/img/checkbox.png" height="15" width="15"> Have a construction process that can be shared.


First thing is <b>Hide the products internal representation from the client</b>
Lets consider what a client is first. It's just the caller for a class right?
We don't need to model that here, its just the top level name space.
A plain function hides its internals well enough to satisfy this condition as
the client doesn't really know what function did or what it returned.

```klipse-cljs
(defn make-pizza [order] order)
```
We update our todo list:

* <img src="/img/checkbox-checked.png" height="15" width="15"> Hide the products internal representation from the client.
* <img src="/img/checkbox.png" height="15" width="15"> Hide how the builder works from the client. 
* <img src="/img/checkbox.png" height="15" width="15"> Have a construction process that can be shared.

Next on the list is, <b>Hide how the builder works from the client.</b>
We might be tempted to say we already did this, but our goal here is to hide the type of builder the client is using. Clojure doesn't have
classes, but it can dispatch to different functions based on data using a multimethod. This way we can <i>order</i> multiple things, like a pizza or a salad.

```klipse-cljs
(defmulti order (fn [{:keys [type]}] type))

(defmethod order "salad" [order] (str "i cant believe you ordered a salad from a pizza place"))

(defmethod order "pizza" [order] (merge order {:order-id 5}))
```

We update our list:

* <img src="/img/checkbox-checked.png" height="15" width="15"> Hide the products internal representation from the client.
* <img src="/img/checkbox-checked.png" height="15" width="15"> Hide how the builder works from the client. 
* <img src="/img/checkbox.png" height="15" width="15"> Have a construction process that can be shared.

Ok now we need to design a  <b>construction process that can be shared</b>. What were constructing is the result of the function. That's just some data,
so we we want to modify it, we can, well just call another function on the return value. Sense the functions are pure and the data is immutable this is safe.

```klipse-cljs
(merge {:coupon "1 dollar off"} (order {:size "medium" :type "pizza"}))
```

update date the list:

* <img src="/img/checkbox-checked.png" height="15" width="15"> Hide the products internal representation from the client.
* <img src="/img/checkbox-checked.png" height="15" width="15"> Hide how the builder works from the client. 
* <img src="/img/checkbox-checked.png" height="15" width="15"> Have a construction process that can be shared.

And were Done. 

###Summery

If you ignore the How of the Builder pattern in the GOF book and look at the <b>Why</b> it comes down to 2 things:

* Hiding data because your data is mutable
* leveraging Polymorphsim 

In clojure our data isn't mutable and we have so few types its simpler to transform our data in the open using core library functions that create
a ritual around it. 

There are a lot of other explanations online that seem to focus on the how something is built up. The ritual of:
```
builder
  .addSomething("thing")
  .addAnotherThing("thing")
```

They say this is helpful because the alternative is having to many constructors for an object and it becoming hard to tell
from the method signature should go where. I have come to favor simple function signatures that just take a map, so to address the above issue would
look like this:


```klipse-cljs
(defmethod order :pizza [order] 
  [{:keys [crust sauce cheese size] :as order}]
  (merge order {:order-id 5}))
```

As a reader i know that keys are being looked for, and as a user of the function i pass my arguments as a map so i there is no confusion 
over what relates to what. After all, did the order of those arguments matter? If not, why are they ordered!

```klipse-cljs
(order {:crust "regular" :sauce "red" :cheese "mozzarella" :size "large" :type "pizza"})
```

Finally someone might quibble that we haven't addressed some of the control over data that we see in other functions such as 
having defaults or checking the arguments range. By passing in the defaults, and adding a pre check, we 
show we can satisfy both of those.

```klipse-cljs
(def pizza-defaults {:crust "regular" :sauce "marinara" :cheese "mozzaralla" :size "medium"})

(defmethod order :pizza [order defaults] 
  [{:keys [crust sauce cheese size] :as order} defaults]
  {:pre [(#{"medium" "small" "large"} size)]}
  (merge defaults order {:order-id 5}))

```

Showing default:
```klipse-cljs
(order {:crust "regular"  :size "large" :type "pizza"})
```

Showing pre-check:
```klipse-cljs
(order {:crust "regular" :type "pizza"})
```

That should throw and error. If its not its because of some mistake i have
made and not a lack of language feature.

