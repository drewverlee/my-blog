{:title "Strategy Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software" "Behavrioral"]}

### Intent

> Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

tldr polymorphism 

### In Clojure

The tldr is that the strategy pattern is designed to achieve runtime polymorphism (as opposed to compile time).

We have covered polymorphism several times in other posts, so i'm to lazy to do it again.
see [run time polymorphic in clojure](https://clojure.org/about/runtime_polymorphism)

if you were originally confused on how polymorphism differs from the strategy pattern
then have a look at this [SO question](https://stackoverflow.com/questions/31608902/polymorphism-vs-strategy-pattern).
If thats not helping you might want to actual take a good look at the man behind the curtain: [static vs dynamic typing](http://www.lispcast.com/static-vs-dynamic-typing)

### Summery

Clojure supports Run time polymorphism.


If your sratching your head about how this differs from the state pattern then i suggest reading 
this [SO](https://stackoverflow.com/questions/18117462/difference-between-state-pattern-and-and-strategy-pattern).
I personally think the differences fall away if you seperate behavior from data (as we do in clojure). 
The state pattern claims to care about state transitions, but if it really did it would make those 
explicit in a state chart. Just having the different states exist and their transitions hidden inside classes is
sloppy.


