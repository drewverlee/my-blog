{:title "Learn Datomic - part 4"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2020-6-6"
 :tags  ["Clojure" "Software" "Database" "Datomic"]}

## Review


in Part two we covered queries e.g

```clj
(d/q '[:find ?e
       :where
       [?e :list/todo _]]
  db)
  ;;ouputs
  [[4178144185548876]]
```


 but queries also come with ability to aggregate 

```clj
  (d/q '[:find (count ?e)
         :where
         [?e :list/todo _]]
    db)

  ;; => [[1]]

```

To make this more interesting lets quickly add some more data

```clj
  (d/transact conn {:tx-data [{:list/title "learning about datomic aggregates"
                               :list/todo  [{:todo/description "talk about how aggregates to speed up performance"}
                                            {:todo/description "talk about `with` function"}
                                            {:todo/description "talk about `with` function"}]}]})

```

A lot of the other aggregates don't make sense given our data
e.g the average of a todo list. You can consult the docs for a full list.

here we see that we can get the number of todo's by list
we show the title but it would work the same if you used the entity id. 

```clj
  (d/q '[:find ?title (count ?todo)
         :where
         [?l :list/title ?title]
         [?l :list/todo ?todo]]
    db)
```

outputs

```clj
 [["learn datomic" 2] ["learning about datomic aggregates" 3]]
```



Here is a bit of a gotcha. We can see from above that the total todo's is 5. So when we count the descriptions
we would expect 5 as well right? because each todo has one description.

```clj
  (d/q '[:find (count ?description)
         :where
         [?l :list/title ?title]
         [?l :list/todo ?todo]
         [?todo :todo/description ?description]]
    db)



```

outputs
```clj

 [[4]]
```

The reason we get back four is because the query function returns a "set" which only has unique values and then takes the count.
we had two todo's with the same description. This is obvious if we include the todo id:

```clj
  (d/q '[:find ?todo ?description
         :where
         [?l :list/title ?title]
         [?l :list/todo ?todo]
         [?todo :todo/description ?description]]
    db)



```

```clj
 [[48910675249987666 "talk about `with` function"] <---- same
    [4178144185548878 "learn how to add the schema"]
    [48910675249987667 "talk about `with` function"] <---- same
    [48910675249987665 "talk about how aggregates to speed up performance"]
    [4178144185548877 "learn how to create a database"]]


```

remove the id


```clj
  (d/q '[:find ?description
         :where
         [?l :list/title ?title]
         [?l :list/todo ?todo]
         [?todo :todo/description ?description]]
    db)



```

```clj
 [["talk about how aggregates to speed up performance"]
    ["learn how to add the schema"]
    ["learn how to create a database"]
    ["talk about `with` function"]] <---- only one
```
If you want to get the count of unique items then use :with

```clj
  (d/q '[:find  (count ?description)
         :with ?todo
         :where
         [?l :list/title ?title]
         [?l :list/todo ?todo]
         [?todo :todo/description ?description]]
    db)
```
```outputs

 [[5]]
```


what `with` does it that it includes the var (e.g todo) in the find specification, like this:

```clj
  (d/q '[:find ?todo ?description
         :where
         [?l :list/title ?title]
         [?l :list/todo ?todo]
         [?todo :todo/description ?description]]
    db)
```
but then drops after the find "part" is complete e.g before the aggregates and before returning. This means the return value
will be a bag (potentially contain duplicates)

Ok one more nifty trick before we wrap it up. 
If instead of returning a tuple you want to return a map, then use the ":keys" key:

```clj
  (d/q '[:find ?title (count ?todo)
         :keys title total
         :where
         [?l :list/title ?title]
         [?l :list/todo ?todo]
         ]
    db)
```
ouputs

```clj

 [{:title "learn datomic", :total 2}]

```


That concludes part 4. Tell next time. 



