{:title "Producer Consumer Problem"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software"]}

<img src="/img/producer-consumer.png">

###Goal

The Goal is to manage multiple threads so we can take advantage of multiple cores,
so our programs can run faster. The "Producer Consumer Pattern" is to design your system
to accommodate this.

At a high level:

* The Producer Group puts meesages onto the datastructure.
* The Consumer Group takes messages off the datastructure.

###Implementation in Clojure

Typically multithreaded solutions involve [mutex locks](https://www.cs.cornell.edu/courses/cs3110/2010fa/lectures/lec18.html) 
under the hood. But I cant advise droping down that low level unless you 
have very good reason. I would venture most problems are better tackled using a higher
[level abstraction](https://dzone.com/articles/producer-consumer-pattern) like a ConcurrentQue, List or array.


Clojure and Functional programming are no different in this regard. Sense Clojure can Interopt with the JVM,
you could take the solution from the link above and implant it clojure. But their is a more native
solution close to hand using a clojure library called [core.async](http://clojure.com/blog/2013/06/28/clojure-core-async-channels.html)

I suggest you take a moment and read at least the rational from the link above. Just to motivate that, here is the opening paragraph:

> There comes a time in all good programs when components or subsystems must
  stop communicating directly with one another. This is often achieved via the
  introduction of queues between the producers of data and the
  consumers/processors of that data. This architectural indirection ensures that
  important decisions can be made with some degree of independence, and leads to
  systems that are easier to understand, manage, monitor and change, and make
  better use of computational resources, etc. 

With that in mind, i think its easy to see how core.async can help us solve the producer consumer problem, it sounds 
like it was created directly to deal with it. Here is my minimal example of how this can be done:

```clojure
;;; we create our channel you can think of it like a que
(def que (async/chan))

;; create a producer that will push some data onto the que
;; you can read  async/>! as "push" so (aysnc/>! que x) would be 'push x onto the que.'

;; the important thing to note is that we call our "push x onto que" function inside 
;; async/go expression, anything inside it runs concurrently and in another thread. 
(defn producer [que x]
  (async/go (aysnc/>! que x)))

;; create a consumer that will pull (<!!) data off.
(defn consumer [que]
  (+ 1 (async/<!! async/que)))

(defn run []
  (producer que 1)
  (consumer que))

;; run the program
(run)
;; result => 2
```

This example is rather lackluster, but it meets our basic requirement. The producer does its work in a different thread
then the consumer. We could easily put the consumer in its own thread. We could also create multiple producers
and consumers. The options are limitless. If you want to read a better example of the producer consumer problem
in clojure look [here](http://elbenshira.com/blog/using-core-async-for-producer-consumer-workflows/).

* For a light hearted into try [clojure for the brave and true](http://www.braveclojure.com/core-async/)
* For a serious look no further then the official [docs](https://github.com/clojure/core.async)

Finally, a word to the wise, don't reach for asynchronsity unless the situation demands it. It comes with a high price in
complexity no matter what abstraction you choose.






