{:title "Learn Datomic Cloud - part 1"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2020-6-6"
 :tags  ["Clojure" "Software" "Database" "Datomic" "Datomic-Cloud" "Application" "AWS"]}

### Datomic Cloud

Datomic cloud is a managed service on AWS that lets us easily deploy your application code 
alongside a persistent storage layer. How easily? Lets see.

Assuming you follow the instructions [here](https://docs.datomic.com/cloud/index.html#org57fb4a4) for a solo topology. 
You end up with a deployed running system that you just need to communicate with. Note some of the configuration below would
be different for a production cluster. A prod cluster is too much money for hobby projects though!

Cutting to the chase, creating an api endpoint looks roughly like this.

1. query database
2. wrap into a return response
3. ionize
4. push to cloud
5. deploy to cloud
6. expose your lambda
7. profit

### 1. Query database

```clj
(defn get-todo-lists
  [db]
  (d/q '[:find (pull ?list [*])
         :where [?list :list/todo ?]]
    db))
```

### 2. wrap into response

```clj
(defn get-todo-lists
  [_]
  (->> "learn-datomic-for-blog"
    utils/get-db
    query/get-todo-lists
    edn/write-str
    edn-response))
```

### 3. ionize

Here we use a library provided by datomic cloud to ionize the function above. 

```clj
(def todo-lists
  (apigw/ionize get-todo-lists))
```

You then reference that function in your ion config as a lambda.


```clj
{:app-name "grow"
 :lambdas
 {:get-todo-lists
  {:fn          grow.ion.http/todo-lists
   :description "lambda proxy integration entry point for todo lists"}
```

### 4. push

```bash
clj -A: '{:op :push}'
```

this captures stores your configuration and code in aws but doesn't run anything. This command will return the deploy
command..

### 5. deploy


```bash
clojure -A:ion-dev '{:op :deploy, :group grow-compute, :rev "<some-id>"}
```

Which will essentially take your stored code that you pushed and makes it a reality on a running computer. 
It's more complex then this of course, read the official docs for more. 


### 6. Expose 

This step is to expose your function as an endpoint and is done through the AWS console. 
Basically from the aws console you do


1. Create API
2. Pick Rest API - BUILD (given what we want)
3. name it "get todo lists"
4. create
5. actions > create resource > configure as proxy > create 
6. set lambda > <system-name>-get-todo-lists
7. settings > add binary type > "*/*"
8. actions > deploy 

### 7. profit

Now curl the end point it sets up...

```bash
curl https://3gr2ovzas1.execute-api.us-east-2.amazonaws.com/dev/datomic
```

And the output should look familiar

```clj
[[{:db/id 4178144185548876,
   :list/title "learn datomic",
   :list/todo
   [{:db/id 4178144185548877,
     :todo/description "learn how to create a database"}
    {:db/id 4178144185548878,
     :todo/description "learn how to add the schema"}]}]
 [{:db/id 48910675249987664,
   :list/title "learning about datomic aggregates",
   :list/todo
   [{:db/id 48910675249987665,
     :todo/description
     "talk about how aggregates to speed up performance"}
    {:db/id 48910675249987666,
     :todo/description "talk about `with` function"}
    {:db/id 48910675249987667,
     :todo/description "talk about `with` function"}]}]]
```


That's it. Given it's possible to an api around one endpoint this can be enough to fuel 
any number of hobby projects. Which my friend [jacob](https://jacobobryant.com/) does to much greater success then me. 
