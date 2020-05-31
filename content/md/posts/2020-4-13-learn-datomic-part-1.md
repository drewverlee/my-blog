{:title "Learn Datomic - part 1"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2020-4-12"
 :tags  ["Clojure" "Software" "Database" "Datomic"]}

### Datomic

Were going to be learning about datomic. This is aimed at beginners.

I don't want to hide anything, so were going to cover the setup. Don't worry though
it's very minimal.


First we setup our namespace and grab the datomic client
library and the edn library which will help read our datomic server 
secrets.

```clj

(ns grow.scratch.learn-datomic
  (:require [datomic.client.api :as d]
            [clojure.edn :as edn]))
```

Just remember that anytime you see d/something that something is 
a datomic api function.

Now let's grab our creds to talk to our datomic service.

```clj
(def client (-> "aws-config.edn"
              slurp
              edn/read-string
              d/client))
```

Now with those creds were gonna go ahead and create a new database. Yes just for you,
thats how effortless this is.

```clj
(d/create-database client {:db-name "learn-datomic-for-blog"})
```

now lets get a connection to our new database

```clj
(def conn (d/connect client {:db-name "learn-datomic-for-blog"}))
```

lets create a schema for our new database for a todo list.
Ignore the details of how this is done

```clj
(def schema [{:db/ident       :list/title
              :db/valueType   :db.type/string
              :db/cardinality :db.cardinality/one}

             {:db/ident       :list/todo
              :db/valueType   :db.type/ref
              :db/cardinality :db.cardinality/many
              :db/isComponent true}

             {:db/ident       :todo/description
              :db/valueType   :db.type/string
              :db/cardinality :db.cardinality/one}])
```


and we transact it

```clj
(d/transact conn {:tx-data schema})
```


Lets add some data a list with two todos

```clj
(d/transact conn {:tx-data [{:list/title "learn datomic"
                             :list/todo [{:todo/description "learn how to create a database"}
                                         {:todo/description "learn how to add the schema"}]}]})
```


wow super meta!

lets go ahead and query the todos in that list. Dont worry about the lets
your eyes pass over this. Dont try to understand it.

```clj
(d/q '[:find ?description
       :where
       [?todo :todo/description ?description]
       [?list :list/todo ?todo]
       [?list :list/title "learn datomic"]]
  (d/db conn))
```

the output is

```clj
[["learn how to add the schema"] ["learn how to create a database"]]
```

rad!

lets break down how this works. 

```clj
(d/q '[:find ?description
       :where
       [?todo :todo/description ?description]
       [?list :list/todo ?todo]
       [?list :list/title "learn datomic"]]
  (d/db conn))
```

first, remove the outer layer (d/q [inner-layer] (d/db conn))
* d/q : is the query function.
* d/db conn : is just the database were doing the query on.

inner layer

```clj
'[:find ?description
  :where
  [?todo :todo/description ?description]
  [?list :list/todo ?todo]
  [?list :list/title "learn datomic"]]
```


the ' tells the compiler to ignore what's next so the function can handle how to read it.
If that doesn't resonate with you no worries, just ignore the it for now.

```clj
[:find ?description
 :where
 [?todo :todo/description ?description]
 [?list :list/todo ?todo]
 [?list :list/title "learn datomic"]]
```

we can break this into two parts

* find 
* where


```clj
:find ?description
```


:find declares what were going to find. In this case a description of a todo
you have to put the question mark before the thing you want to find.

:where tells us where to find the thing! Lets read this line by line

we want the ?description for the ?todo

```clj
[?todo :todo/description ?description]
```

where that ?todo is part of a ?list

```clj
[?list :list/todo ?todo]
```

and that ?list title is "learn datomic"

```clj
[?list :list/title "learn datomic"]
```

great we did it!
That a major chunk of what datomic is about.. getting information.

I'm proud of you. Join me next time as we learn a bit more about datomic!
