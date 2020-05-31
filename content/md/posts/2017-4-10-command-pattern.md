{:title "Commander Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software"]}

###Goal
Commander Pattern is about controlling execution of a function on some data.

###Implementation in clojure

A command holds everything we need to execute a function on some data. This command
holds the addition operator and a list of arguments `[1 2]` that we plan on adding together:


```klipse-cljs
(def command {:cmd + :args [1 2]})
```

Just-execute is a function that true to its name, just executes the command. Don't worry
about how this is done:

```klipse-cljs
(defn just-execute [{:keys [cmd args] :as command}]
  (apply cmd args))
```
invoke it and as you expect the result is 3

```klipse-cljs
(just-execute command)
```

We have demonstrated the benefits the commander pattern tries to achieve. To demonstrate this point further consider
that we can now do anything we like with this command. For example store it somewhere:

```klipse-cljs
(defn store-and-execute [store {:keys [cmd args] :as command}]
    (conj store (assoc command :result (apply cmd args))))
```

invoke it (the output will be hard to read as its returning the javascript function equilvent)
```klipse-cljs

(store-and-execute [] command)
```

Or maybe we want to add another number only before we before the operation:

```klipse-cljs
(defn add-number-and-execute [{:keys [cmd args] :as command}]
    (apply cmd (conj args 5)))
```

We get 8 as we expect:
```klipse-cljs
(add-number-and-execute command)
```

We can do anything we want with the function and this data. We have complete control and so have satisfied the Commander Pattern.

###Summery:
The Commander Pattern is about controlling when and where a function is called and on what. This sounds like it should
be a fundamental part of a programming language, and for the most part it is. Nearly all languages give you this ability,
but to what degree is another matter. Any language without fully supported first class functions is going to leave
its practitioners scratching their heads on how to retro-fit them in. This is why we have the Commander Pattern, because 
its confusing enough and hard enough to achieve this level of flexibility that its worth giving a name. In short,
the Command Pattern is a work around for the lack of first class functions. 

###Explanation:

The following will only make sense if you have read the literature on the Commander Pattern.

So how does having first class functions remove all the abstractions we typically find in the Commander Pattern?

I wont produce another UML chart here, I encourage you to go find some. But here are the abstractions/classes:

* Commander Interface/Command
* Commander Implementation/Concrete Command
* Receiver
* Client
* Command
* Invoker

There seems to be a lot going on here, and their is, there is a lot going on in the code in the Implementation
section above, but the point is you didn't have to do any of the work. 

Sense in we can pass functions around , there is no need to have a Command Interface or its Implementation
to accommodate this ability to call a method generically so it just becomes part of the Invoker (`just-execute`).

The Reciever is just the original function we want to control (`+`). 

The client took a reference to the function and called it given the Invoker and the receiver.  In our
example it just becomes where-ever we choose to call `just-exucte` against the command. We don't need to put those
into a special class or namespace.


