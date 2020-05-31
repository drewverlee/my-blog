{:title "Simple By Design"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "tutorial"]}


### the Site is Interactive

Go ahead and change the words below:

```klipse-cljs
"Make the wrong thing hard and the right thing easy, every other
plan will fail under stress."
```


This presentation is interactive. So you can try out the code, or modify it. Have Fun!

If you want a better online editor go [here](http://clojurescript.io/). If you get
stuck on something related to Clojure try the Clorian Slack #Beginners channel.

### On discussing Programming languages

I was discussing programming languages on the Michigan Devs slack and someone said
something very profound on the topic of discussing programming languages:

>  I believe the human tendency is to distill an important yet complex topic down
>  to simple practical conclusions, and become emotionally attached to those
>  conclusions.  This is necessary in order to make confident high-level decisions
>  on what to do about the topic within the contexts that we live, and defend
>  those decisions later.  As a result, perhaps the hardest thing to find in
>  programming literature and communities are perspectives and ideas that are both
>  reasonably objective and still useful within any single context.  The diversity
>  in programming environments and business contexts is so vast, that it is indeed
>  very challenging to make any statement of wisdom that is broadly applicable.  I
>  think high-level discussions about programming languages and paradigms are very
>  clear examples of this unfortunate dynamic.

- Jerry Wiltse

I think Jerry has touched on something very important, that many of our
decisions are made out of necessity. Many factors play into our choice of programming
language:

* prior knowledge
* the task at hand
* hiring power

Are just some of the considerations for what the "best language" is. So
in reality, "best" is high subjective. So rather then argue about which
language is best, i would rather talk about why I enjoy Clojure.

# Why I enjoy Clojure

<img src="/img/Clojure_logo.svg">


I enjoy Clojure because it helps me keep things simple. 

## What is simple?

There are a couple definitions but the one i like the most is:

> Not Compounded 

Compounded should summon images like this:

<img src="/img/compound.gif">

Of something entailed and intertwined, think "spaghetti code". You should
have a slightly bad taste in your mouth.

## what isn't simple? (but still valuable)

* Convenience - its easy to get up and going (Python is installed on computers)
* Beginner-friendliness - warm and friendly (ruby works very hard at this)
* Ease - is relative, for Lebrone James Dunking is easy
* Familiarity - important for adaption
* Less steps - less isn't necessary better if it means mushing things together which can lead to confusion.

## Why is Simple valuable?

> It can scarcely be denied that the supreme goal of all theory is to make the
> irreducible basic elements as simple and as few as possible without having to
> surrender the adequate representation of a single datum of experience.

Which is often paraphrased as:

>  Everything should be made as simple as possible, but no simpler.

 - Albert Einstein

## How Clojure enables Simplicity:

* Simplicity of Syntax
* Simplicity of Protocols
* Simplicity of Values and Reference

#Syntax
## Hello World in Clojure:

```klipse-cljs
"Hello World"
```

This is a full program. It has complied just for your viewing pleasure. This is possible because
everything is Data. What we have below is a hash-map `{}`:

## Simple Primitives

There are a small set of pieces from which we construct things in Clojure.
These are displayed below in a map:

```klipse-cljs
{
 :string "hello"
 :character \f
 :integer 42
 :floating-point 3.14
 :boolean true
 :symbol +
 :keyword [:foo ::foo]
 }
```

Notice the symbol is displayed as the javascript function it represents.

You might be curious about keywords. Keywords are names and look themselves up in maps.
In this, and a couple other ways, they are different from Strings.

You can also discover the type of something by doing asking for it using the `type` function.

```klipse-cljs
(type ::test)
```

Keywords with Two colons means its a fully qualified keyword (its unique in the whole wide world):

```klipse-cljs
::test
```

As opposed to a keyword with one colon, which you will notice doesn't have the full namespace `clj.user`

```klipse-cljs
:test
```

## Collections

Here is a map of the collection types, one of which is a map. (how meta)

```klipse-cljs
{:list '(1 2 3)
 :vector [1 2 3]
 :map {:a 1 :b 2 :c 3}
}
```

The `'` on the list might have caught your eye. This tells the
clojure to not evaluate the list. Why is that problem? Good question!
Lets see...

```klipse-cljs
(1 2 3)
```

If you scroll through that error message you will see;

> TypeError: 1 is not a function

In Clojure the first argument to a list needs to be a function:

```klipse-cljs
(+ 1 2 3)
```
## Function call semantics

<img src="/img/language-semantics.jpg">

## You know now all the syntax of Clojure

Yes. Really. Clojure programs are written using those just those primitives
and that one simple rule <i>language designers dont want you to know about</i>.

### Clojure Functions

Lets look at a function that makes other functions `defn` which you can
read as "define function".

```klipse-cljs
(defn greet
  "returns a friendly greeting"
  [your-name]
  (str "Hello, " your-name))
```

Go ahead and try it out:

```klipse-cljs
(greet "Chris Broski, the one true Chris!!!")
```

Lets look at this in detail:

<img src="/img/clojure-functions.jpg">

So you can see that we built this function using our core primitives:


<img src="/img/clojure-function-types.jpg">

# Macros

Macros allow you to extend the language. This means you can make a DSL quite easily.

```klipse-cljs
;; ignore the namespace
(ns my.m$macros)

(defmacro backwards
  [form]
  (reverse form))
```

```klipse-cljs
(my.m/backwards (" backwards" " am" "I" str))
```

This macro `backwards` intercepted the form `(" backwards" " am" "I" str)` and reversed it
to  `(str "I" "am " "backwards")` before that form evaluated. This is important, because
as we discussed The operator must be the first thing in the form/list. See what happens without the macro?

```klipse-cljs
(" backwards" " am" "I" str)
```

This is made possible because Clojure is a Lisp and so you as a programer can manipulate both
the regular form (what any language can do) but also the form created the parser and lexer:

<img src="/img/lisp-eval.png">

Generally speaking, you should *strongly* prefer functions over macros. 
However their are times when using a macro can greatly simplify a problem. Here is a [blog
post](http://www.lispcast.com/when-to-use-a-macro) about when to use macros if
you want to learn more. But the three main situations are:

* Gaining control over the control structure (we demoed this above)
* Performing compile-time computation
* Providing custom syntax

## Simplicity of Syntax recap:

* Not much syntax
    - Personal I have noticed I can do far less with more in langauges like Ruby and Python. Whose constructs seem more limited, and harder to remember
* Not much about ordering
    - This goes in the middle, this goes at the top, etc..

* A preference for re-using the same data structures.
    - This is hard to show the benefit from in a small example. It comes from their being a very rich library of functions
    in clojure that can work against these data structures, which allows the developer to focus on learning how to manipulate data.

* Macros allowing for powerful language manipulation.


### Simplicity in Protocols

A Protocol is :

> a set of rules governing the exchange or transmission of data between two services.

Lets work through a quick example. Say we have three Reversible things.

<img src="/img/object-design-1.jpg">

In this case our protocol is the Ability to reverse things and we need a way set of rules to govern
how different services/types are reversed.

How would i go about modeling this in a OO language?
Possible something like this:

<img src="/img/object-design-2.jpg">

The Interface abstraction in Java would Interface.
The Implementation abstraction in Java would be a Class.

However it seems like something isn't labeled on our Model, the arrows to and from our interface:

<img src="/img/object-design-3.jpg">

What is that in Java? Or Ruby? Or javascript?  It's an interesting question, and will
go into it more an just a bit.

In java this often times it involves some sort of design pattern.
For instance, I believe the Adaptor pattern might help:

<img src="/img/ClassAdapter.png"

>  Class Adapter pattern

> This adapter pattern uses multiple polymorphic interfaces implementing or
> inheriting both the interface that is expected and the interface that is
> pre-existing. It is typical for the expected interface to be created as a
> pure interface class, especially in languages such as Java (before jdk 1.8)
> that do not support multiple inheritance of classes.[1]

If were in a dynamic language like Ruby and Python we might be able to monkey
patch, but the patchs might overlap and cause problems.

### How do we simplify this?

To make something Simpler, identify the part of the Model that has no
specification and give it one. Put another way, if your
Model has more things then names, its compounded to some degree.

In Clojure(script) we have an abstraction to extend a protocol, its simple ccalled `extend-protocal`.

### Extending protocals

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

### Polymorhism in Clojure

What we just learned was Polymorhism in Clojure. How to have one function
`reverse` behave differently on different types. Unlike in most OO langauges we
have an additional abstraction `extend` which allows us to extend existing classes.

What we avoided by having this:

* adapter pattern
* wrapper pattern
* translator pattern
* monkey patching
* StringUtils

## Simplicity in values and references

Lets say we want to model me at my highschool weight:

<img src="/img/clojure-simple-by-design-27-638.jpg">

And the transition to my current weight...

<img src="/img/clojure-simple-by-design-29-638.jpg">

In OO languages we might do something like this.

<img src="/img/clojure-simple-by-design-30.png">

These question marks represent the idea that typically we only
capture the most current value of something. So in terms of labeling things
were forced to re-use an idea and leave others blank.

### Simplify by addressing the missing pieces:

<img src="/img/clojure-simple-by-design-32-638.jpg">

Clojure has a extremely rich set of built in tools to handle
state management when its really necessary. Here were going to use
whats called an atom.

```klipse-cljs
(defprotocol Nachos
 (yum [_] "eat some nachos"))

 (defrecord Person [name lbs]
   Nachos
  (yum [person]
        (update-in person [:lbs] + 2)))

(def me (atom (->Person "Drew" 182)))

(def me-before @me)

(swap! me yum)

(def me-after @me)

me-after
```

Were able to isolate the functional part of the program..

<img src="/img/clojure-simple-by-design-34-638.jpg">

From the stateful part which requires update semantics

<img src="/img/clojure-simple-by-design-35-638.jpg">

And  effectively model the transition.

<img src="/img/clojure-simple-by-design-36-638.jpg">

The key element here isn't that we captured before and after,
we could do that easily with varibles in most languages. The
key is that Atom is designed to work intelligently if their
are multiple threads accessing that information.

When we call Swap on an atom, if that atom has been changed by the time
were done applying the function to it, then we re-try with the new value:

<img src="/img/atom-swap.png">

###Complexies avoided:

* Lack of temporal reasoning
* Single threaded program
* Locking
* Defensive copying
* Setter methods
* Str vs StringBuilder vs StringBuffer

And its important to note that this problems can easily become combinatorial,
that is, compound with each other.

### This is all built into clojure.

So what jumps out at you when look at these various solutions to the dinning
philosphers problems?

* [java](https://rosettacode.org/wiki/Dining_philosophers#Java)
* [clojure](https://rosettacode.org/wiki/Dining_philosophers#Clojure)
* [ruby](https://rosettacode.org/wiki/Dining_philosophers#Ruby)

Clojure doesn't need any import statements! I'm not arguing this is better,
but it's an important point that Concurrency was built into
Clojure from the [start](https://clojure.org/about/state).


### Benefits of clojure

So we have talked about how clojure enables simplify but its also useful
to frame clojure in terms of what it enables you todo. Will look at some
examples and see how they match to some high level themes:

* <b>Concision</b> : shorter programs, which all things equal, is good
* Generality       : can handle a number of different situations
* Robustness       : program correctness (no surprises)
* Agility          : how easy it is to change a program.

To understand how Clojure enables concision lets look at a simple of 
a function `indexOfAny` that find the index of the first character in
a string given a set of strings. Here is the specification:

<img src="/img/clojure-simple-by-design-41-638.jpg">

Were going to walk this code through a series of transformations in order
to make it into a clojure program.

<img src="/img/clojure-simple-by-design-42-638.jpg">

First we remove the corner cases, mainly returning -1 (which doesn't mean anything)
and instead returning an empty list (which will show later):

<img src="/img/clojure-simple-by-design-43-638.jpg">

Then we remove the type declarations:
<img src="/img/clojure-simple-by-design-44-638.jpg">

Leverage the when function:

<img src="/img/clojure-simple-by-design-45-638.jpg">

Leverage list comprehension: 

<img src="/img/clojure-simple-by-design-46-638.jpg">

Use clojure syntax:

<img src="/img/clojure-simple-by-design-47-638.jpg">


Looks like left ouf the implementation of indexed (unless im missing a built in function somewhere...)

```klipse-cljs
(defn indexed [coll]
  (map-indexed (fn [idx itm] [idx itm]) coll))

(defn index-filter [pred coll]
    (for [[idx elt] (indexed coll)  :when (pred elt)] idx))
```

### Benefits of clojure

* concision
* <b>generality</b>
* robustness
* agility

<img src="/img/clojure-simple-by-design-49-638.jpg">

get indexs of heads in a stream of coin clips...

```klipse-cljs
(index-filter #{:h}
  [:t :h :t :h :t :h :h :t :t :t])
```
Or maybe the get the index of any fibonaccis number that is less then 10.

Make a quick fib generator:
```klipse-cljs
(def fib-seq
  ((fn rfib [a b]
     (lazy-seq (cons a (rfib b (+ a b)))))
   0 1))

(take 20 fib-seq)
```

Find the index of those less then 10

```klipse-cljs
(index-filter #(> % 10) (take 20 fib-seq))

```

### Generality

So what was a 40 line program that only worked on strings is now
a 4-5 line program that works on anything. This type of shortening
_nearly_ as well in another dynamic language like Python, Ruby, Javascript.

But i would argue Clojures has less language constructs then many of the other
dynamic languages i have used and so tends to have better re-use of abstractions
and functions which leads to smaller and more perform ant solutions overall.  This is because
clojure is made up of PICOs

### Plain Immutable collection objects PICOS.

While in many other languages you have a multiplication of types
each with their own functions. In clojure, in stead of the developer defining
new classes you just have:

* <b>P</b>lain: lists, maps, sets.
* <b>I</b>immutable: less defensive code.
* <b>C</b>ollections: tons of existing (highly performant) functions already work on them.
* <i>Objects</i>

### And we model everything in clojure this way:

* collections
* directories
* files
* XML
* JSON
* result sets
* web requests
* web responses
* sessions

Which means you can do things like model the HTML

```klipse-reagent
(require '[reagent.core :as r])

(defn simple-component []
  [:div
   [:p "I am a component!"]
   [:p.someclass
    "I have " [:strong "bold"]
    [:span {:style {:color "red"}} " and red "] "text."]])

(defn simple-parent [name]
  [:div
   [:b "my name is: " name]
   [:p "I include simple-component."]
   [simple-component]])

[simple-parent "drew"]
```

### Generality through re-use:

Lets filter something

```klipse-cljs

(filter odd?)
```

Then take increment it..

```klipse-cljs
(map inc)
```

Lets roll that into one a variable (`def` creates a global variable)

```klipse-cljs
(def xf (comp (filter odd?) (map inc)))
```

What should be of interest to you at this point is that we haven't worried about the inputs and outputs.
Whats amazing is that our code as written above will work on:

* collections
* Streams
* channels
* observables

Here its is working on Collections:

```klipse-cljs
(transduce xf + (range 5))
```

### Generality in persistence:

In order to build real services we need more then just a programming language, we 
need a persistent store. Of course Clojure can use any database (just like any language) but it 
has a lot of harmony with a particular database called Datomic. There is a *lot* we could say about it, but 
what i want to focus on is how you interact with it when programming. 

Here is a datalog query (which is the query language of datomic):

```
[:find ?title
 :where
 [_ :movie/title ?title]]
``` 

```
# results...
"First Blood"
"Terminator 2: Judgment Day"
"The Terminator"
"Rambo III"
"Predator 2"
"Lethal Weapon"
"Lethal Weapon 2"
"Lethal Weapon 3"
"Alien"
"Aliens"
```

Dont' worry about how the query langauge works. Whats awesome is that this query is built using the same primitives as the rest of the language. Particularly its written in extensible data notation (edn)
which is similar to Json, but it:

* Is extensible with user defined value types
* Has more base types
* Is a subset of clojure data

### Benefits of clojure

* concision
* generality
* <b>robustness</b>
* agility

<img src="/img/clojure-simple-by-design-53-638.jpg">

The result is that our programs are more robust and easier to understand
and potentially faster because we let the language handle lower level
details.


### Safety where you need it.

Lets say your work for a very important business whose API accepts
Big Even Numbers from important discerning clients.

First some grunt work to pull in the required libraries:

```klipse-cljs
(ns my.spec
  (:require [clojure.spec.alpha :as s]
            [clojure.spec.gen.alpha :as gen]))

```

Now we define our big even specification:

```klipse-cljs
(s/def ::big-even (s/and integer? even? #(> % 1000)))
```

We confirm it works..

```klipse-cljs
(s/valid? ::big-even 10)
```

```klipse-cljs
(s/valid? ::big-even 100000)
```

Now we can attach this to our API and catch any nasty none big numbers
that try to get in...

```klipse-cljs
(s/explain-data ::big-even 5)
```

And just as important, we can use that specification to generator as many big
even numbers as we want and use them as inputs.

```
(gen/sample ::big-even )
;;=> (3202 4502 3814 8902 20206)
```


<i> clojurescript doesnt currently support generators</i>


### Robustness in Clojure

* Less variability in a program means less to go wrong
* Specify not only the types, but the acceptable range and composition of those types
* Increased Confidence in our program through Property Testing

### Benefits of clojure

* concision
* generality
* robustness
* <b>agility</b>

The arguments for  agility in clojure is that we have already achieved it
with this by satisfying the last three.

### Next Steps

<i>Clojure for the Brave and true also has install steps</i>

1. [Install Clojure for MAC](http://www.lispcast.com/clojure-mac)
2. Edit code with [Atom](https://gist.github.com/jasongilman/d1f70507bed021b48625), [Emacs](http://www.braveclojure.com/basic-emacs/) or [Intelliji](https://cursive-ide.com/)
3. Profit?
4. choose a path:

Fun:
------------
* [Clojure for the Brave and True](http://www.braveclojure.com/) or [living clojure](http://shop.oreilly.com/product/0636920034292.do)
* [Koans](http://clojurekoans.com/)
* [CLojure applied](https://pragprog.com/book/vmclojeco/clojure-applied)
* Build Stuff

Clojurescript:
-------------
* [Clojure for the Brave and True](http://www.braveclojure.com/) or [living clojure](http://shop.oreilly.com/product/0636920034292.do) or [clojure unravled](https://funcool.github.io/clojurescript-unraveled/)
* [Koans](http://clojurescriptkoans.com/)
* [CLojure applied](https://pragprog.com/book/vmclojeco/clojure-applied)
* Build Stuff

Hurt Me Plenty:
-------------
* [joy of clojure](https://www.manning.com/books/the-joy-of-clojure-second-edition)
* [CLojure applied](https://pragprog.com/book/vmclojeco/clojure-applied)
* Build Stuff


### Credit

* <i>Most of the  content is from "Simplicity by design" by Staurt Halloway.
All credit goes to him.</i>
* The section on Macros is From Clojure Brave and True
* Klipse allows the site to be interactive
* http://blog.klipse.tech/clojure/2016/05/30/spec.html



