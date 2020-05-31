{:title "Observer Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software" "Behavrioral"]}

### Intent

> Define a one-to-many dependency between objects so that when one object
> changes state, all its dependents are notified and updated automatically.

That intent is actual very straight forward, put another way, when the
state of A updates, the state of B should update to.

### In Clojure

Clojure has several stateful abstractions built in that handle concurrency issues
so you don't have to. Lets use a helpful chart to pick the right one:

<img src="/img/eB3DR.png">

Look at all the questions you need to answer in order to adequately handle
real state coordination. None of which are address in the observer pattern.

I create a toy example and choose an abstraction based on that, but
it would be pedantic. 

### Summery

Different ways of creating Observers are built into Clojure.
