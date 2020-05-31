{:title "Learn Datomic - part 2"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2020-4-18"
 :tags  ["Clojure" "Software" "Database" "Datomic"]}


## Review 

In part one we created a database and learned to use the datomic query language to get some information.
In Datomic talk, that information is a set of "facts".

Our query:


```clj
[?todo :todo/description ?description]
[?list :list/todo ?todo]
[?list :list/title "learn datomic"]
```

was made up of "clauses". Here is a single clause:

```clj
[?todo :todo/description ?description]
```
and used "variables" which are prefixed with a ? e.g todo
to unify our results. e.g In the case below ?todo "binds" these
two clauses.

```clj
[?todo :todo/description ?description]
[?list :list/todo ?todo]
```
as opposed to if we only had the clause

```clj
[?todo :todo/description ?description]
```

we wouldn't know what list's they were from.

This way of querying data is using datomi's Query API which is 
based on datalog. I highly reccomend this [talk](https://www.youtube.com/watch?v=8rRzESy0X2k) for learning more
about how datalog, sql and rules engies relate to each other.

## Going further

But wait theres more! Datomic has another way to query data
called the "PULL" basically it means you give it the root
and it will return the tree. If you have every used a document store
it behaves like that.

We already did that in part one. When we attached db/isComponent true to the list's todo

```clj
(def schema [{:db/ident       :list/title
              :db/valueType   :db.type/string
              :db/cardinality :db.cardinality/one}

             {:db/ident       :list/todo
              :db/valueType   :db.type/ref
              :db/cardinality :db.cardinality/many
              right here 
              :db/isComponent true}

             {:db/ident       :todo/description
              :db/valueType   :db.type/string
              :db/cardinality :db.cardinality/one}])
```


lets see a pull in action
```clj
(d/q '[:find (pull  ?list [*])
       :where
       [?list :list/title "learn datomic"]]
  (d/db conn))
```

the output:

```clj
[[{:db/id 4178144185548876,
   :list/title "learn datomic",
   :list/todo
   [{:db/id 4178144185548877,
     :todo/description "learn how to create a database"}
    {:db/id 4178144185548878,
     :todo/description "learn how to add the schema"}]}]]
```

see we got a tree with the list as the root and the todos as leaves.
This is a great way to get get information that has been grouped together.

That's it for today. Go outside, but keep away from everyone! #2020
