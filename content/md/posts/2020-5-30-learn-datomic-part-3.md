{:title "Learn Datomic - part 3"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2020-5-30"
 :tags  ["Clojure" "Software" "Database" "Datomic"]}


## Review 

In part two we explored how datomic could mimic a document store. e.g

```clj
(d/q '[:find (pull  ?list [*])
       :where
       [?list :list/title "learn datomic"]]
  (d/db conn))

```

## Onward

Now its worth exploring that this pull concept exists outside of query.
to use the datomic pull api directly we need 3 things which have been introduced before

1. the db
2. the pull pattern
3. the entity id

lets get the entity id for a list

```clj
(d/q '[:find ?e
       :where
       [?e :list/title _]]
  db)
```

outputs

```
[[4178144185548876]]
```

now we can use that id with the pull pattern '*' which means pull everything.

```clj
(d/pull db '[*] 4178144185548876)
```

outputs

```clj
 {:db/id 4178144185548876,
     :list/title "learn datomic",
     :list/todo
     [{:db/id 4178144185548877, :todo/description "learn how to create a database"}
      {:db/id 4178144185548878, :todo/description "learn how to add the schema"}]}

```

of course you might not need everything. so lets try a more specific pattern

```clj

(d/pull db '[{:list/todo [:todo/description]}] 4178144185548876)
```
outputs

```clj
#:list{:todo
           [#:todo{:description "learn how to create a database"}
            #:todo{:description "learn how to add the schema"}]}

```
Ids are hard to remember though, this isn't a problem for our computer friends but we might want something
more human friendly when developing. good news, Datomic has a way for you to make an [attribute value] pair unique.
Once unique, if you specify that pair then you will update rather then create a new entity. Lets demonstrate that
by adding another list with the same title as an existing list.

```clj
(d/transact conn {:tx-data [{:list/title "learn datomic"}]})
```

Check the database

```clj
(d/q '[:find ?e ?t
       :where [?e :list/title ?t]]
  (d/db conn))
```

outputs

```clj
[[4178144185548876 "learn datomic"] [25244786973737039 "learn datomic"]]  
```
 see their are two entities with that title. it created a new one. Once we lock
 down the title that will be impossible. In fact the command below to
 make the title unique will fail tell we the title is unique



```clj
(d/transact conn {:tx-data [{:db/ident  :list/title
                             :db/unique :db.unique/identity}]})
```

outputs an error. Lets change the title so there all unique.


```clj
(d/transact conn {:tx-data [{:db/id      25244786973737039
                             :list/title "revenge of the datomic"}]})
```

 now we can pull using our new unique [attribute value] by specifying  :db/unique :db.unique/identity


```clj
(d/transact conn {:tx-data [{:db/ident  :list/title
                             :db/unique :db.unique/identity}]})
```

;; Now we can pull using our new unique identity!


```clj
(d/pull db '[{:list/todo [:todo/description]}] [:list/title "learn datomic"])
```

outputs:

```clj
 #:list{:todo
           [#:todo{:description "learn how to create a database"}
            #:todo{:description "learn how to add the schema"}]}


```

This is great. Why isn't it the default? my guess would be because it
requires more book keeping by the database so it should be used mostly
as a developer friendly way to work and inspect that data. 


Thats it for this bite of datomic. tell next time!
 

 end part 3

