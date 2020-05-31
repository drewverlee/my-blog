{:title "Bridge Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software"]}

<img src="/img/bridge.jpg">

###Goal

Bridge pattern is described in the GOF as:

> decoupling an abstraction from its implementationso that the two can vary independently**  

I suggest reading this [Explanation](http://www.journaldev.com/1491/bridge-design-pattern-java).


However, i think i haven't read a single description that actual captures what their doing 
in practice, so i put foth my own:

<b>The Bridge pattern is about  extending data independently of the functions that operate on that 
data</b> 

###Implementation in clojure

Lets motivate a problem first. We have shapes and colors and we want to draw them.
<i>I know, your staggering at the task already</i>

First unlike, 90% of the examples on the web, i'm going to do the sane thing and say that
you can't draw a shapeless color or a colorless shape. Both are nonsense. It's going to take us a while to fullfil
that requirement, but its very satisfying when we do. 

This is what our api would look like if it were built:

`(draw drawable)`

Whats a drawable you say? Well, its a thing that can be drawn and it can be drawn
if it has both a shape and a color.

lets go ahead and define drawable using a multimethod.  A multimethod defines a
function with a name (`drawable`).  When the function is called `(drawable arguments)` the results determine the actual function the arguments will be
called on. Hold onto your questions, you already understand what this is doing
from that description but you need to see it in action to be sure.

```klipse-cljs
(defmulti draw (fn [{:keys [color shape]}] [color shape]))
```

Now we define a defmethod, which is the  function that the multimethod might pass 
the arguments it got on to. 

```klipse-cljs
(defmethod draw ["blue" "circle"]  [{:keys [color shape]}]
          (str "drawing a " color " " shape "!"))
```

If it helps you can think of the multimethod like a rail-road switch.

<img src="/img/rail-road-switch.jpg">

With the defmethods being the tracks afterwards, the train here would be the arguments themselves e.g `{:shape "circle" :color pink}`. The switch decides
which defmethod to use. 

Lets return to the problem at hand and draw a blue circle:

```klipse-cljs
(draw {:shape "circle" :color "blue"})
```

We have demonstrated the ability to extend data independently of the functions that operate
on them and so have satisfied the Bridge Design Pattern.

###Summery:

The Bridge Pattern is notable because it decouples abstractions, so they can be
composed together.  It allows a shape type to take a color type without it
needed to now how colors work. This follows the wisdom of "preferring composition over inheritance"  It should also remind you of a SOLID principle.
Take a moment a think of which one... The answer is [Open/Close Principle ](http://joelabrahamsson.com/a-simple-example-of-the-openclosed-principle/)
It might be worth your while to work through that blog post, it solves nearly
the same problem that the Bridge design pattern is trying to address.
Ironically though, the solution in the Open Closed Principle blog post fails to
address the issue of what happens when we add another Function like `Find
paraimter` Now were back to opening up every class to make this change. This
problem has a name, its called the [ExpressionProblem ](https://www.ibm.com/developerworks/library/j-clojure-protocols/). If that
link didn't tip you off, clojure solves this using multimethods, which is what we 
leveraged above in our implementation, because again the Bridge Design pattern is related to the open closed 
principle in that they both suggest you compose your functionality together, rather then build a ridge hierarchy (inheritance).

###Explanation:

Multimethods solve the problem because they can dispatch on a function over the data and sense a function over data
can do anything, this gives you unlimited flexibility in decoupling the data your dispatching on. I'm willing to
bet we solve a lot of design patterns with multimethods. Let's return to another part of the OO solution that 
doesn't really jive well with the goal of being flexible. Why does a shape _have_ a color. Why doesn't a _color_ have a 
shape. The reality is both ways of thinking about the problem are a distraction.  And if you go back to our 
implementation of a drawable, we didn't (think about it..):

```klipse-cljs
(draw {:shape "circle" :color "blue"})
```

We passed a map with a shape and color key. The map itself isn't a shape or a color. It simple holds that information, were
able to draw it because it contains things that when combine together are ... drawable. So what makes something drawable isn't its type,
but the data it holds. If something doesn't hold a shape or color key, its not draw-able. 
Which reminds me, we didn't complete our interface, we said it wasn't possible to draw a shapeless color. We never checked
to make sure that was true:

```klipse-cljs
(draw {:color "blue"})
```

So that throws an error. Which isn't quite right, we should handle that case.
Also what if we define a new shape and color that we don't have  dispatch value on? What if we want some default value?

```klipse-cljs
(draw {:color "pink" :shape "square"})
```
Also an error.

Luckly the solution to both these woes is to have a default value:


```klipse-cljs
(defmethod draw :default [{:keys [color shape]}]
  (if-let [attr (and color shape)]
    (str "drawing a " color " " shape "!" )
    (str "Ugh i can't draw that")))
```

Now we can draw any new color & shape combination with no changes to the data itself.
```klipse-cljs
(draw {:color "green" :shape "Hulk"})
```

And we elegantly handle when we can't draw something.

```klipse-cljs
(draw {:color "green"})
```

There we have it, the itch the Bridge Pattern is trying to scratch is handled elegantly by a feature in Clojure.

###Extra Credit

Reality sets in and you realize that you <b>can't draw something if it doesn't have a size!!!</b>
Use your new found super powers to address this issue. Be careful and think about:

* What parts need to changes to accommodate this?
* breaking backward compatibility with existing code
* should i create a new method or just change the old one?


```klipse-cljs
;; TYPE HERE (it auto evaluates your code)
```



