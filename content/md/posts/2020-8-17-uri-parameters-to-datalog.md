{:title "Considering alternatives to the CommonURLParameters Language"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2020-8-17"
 :tags  ["Clojure" "Software" "datalog" "webapplications" "datomic"]}

## Introduction

<img src="/img/sp.png">

The above is an example program written in what i'll be calling the CommonUrlParmeters Language (CUP).
which belongs in a family of UrlParameters languages. If this example looks like
a typically browser client URL your correct. The purpose of this post is to explore what 
falls out of thinking of the url as a programming language.


## History 

A brief look into the history of the url tells us that:

1. Path parameters were based off the unix file system.
2. Query parameters were originally used to search index's and were later repurposed for webforms.

Given the historical drive to evolve the url was driving by increasing communication demands, it should be natural
to consider what the current demands are and if the CUP language or any langauge in the Url Parameters family is a good fit.

## Where are we now?

The current internet landscape is massive and the competition is greater then ever,
This has been pushing the people to express arbitrarily complex functionality in 
these languages. Consider this healthcare specification called fhir:

###  [composite URL](https://www.hl7.org/fhir/search.html#combining)

 > GET [base]/DiagnosticReport?result.code-value-quantity=http://loinc.org|2823-3$gt5.4|http://unitsofmeasure.org|mmol/L
 
 Without going into details it should be clear that there is more going on here then your typical webapi. This string needs to be parsed by a specific grammar and translated into a language your system and eventually your database needs to understand. As a developer you should be as concerned about the composability and simplicity of your URL language as your database query language.
 
## Unifying concepts in order to compare.

I find i learn best by connecting new ideas to existing ones that i have a stronger mental model. To that end,
I put forth that the logic a back-end server employs to turn url parameters into 
executable code is the same process that a compiler for a traditional
programming language takes:


1. Tokenization : Extracting the parameters and values from the url parameters.
2. Lexical analysis : Defining the relationship between the  path parameters and query parameters.
3. Emitter: Turning the relationship into executable code.

To demonstrate this lets write a compiler for a language within the
UrlParamete's family. This wont be the CommonUrlParmeters langauge (CUP),
but it will be surprisingly close for very little work. Will be emitting a valid
query map that could be passed to the datomic query api.

To simplify were going to treat the path parameters the same way query parameters are usually handled.
You can think of a set which contains all your data and each key value pair as a filter on that set.

For example. Give me everyone named sam who is male might look like like this:

```
"name/sam?gender=male"
```

The Grammar only needs to distinguish between are attributes (keys) and their values:

```
QUERY = CLAUSE+
CLAUSE = (ATTRIBUTE <S> VALUE [<S>])
S = "/" | "&" | "?" | "="
ATTRIBUTE = WORD 
VALUE = WORD
<WORD> = #'[a-zA-Z]+'
```

If we walk our parse tree we see that we have a query made up of clauses which are made of our attribute value pairs:

<img src="/img/ssp.png">

The final step is to emit/transform our tree into something which can be executed. In our case, 
that means transforming it to a [datomic query map](https://docs.datomic.com/on-prem/clojure/index.html#datomic.api/query) which is the shape you pass to the datomic query function. The simplest way is to walk up the tree and transform each type. 

```clojure
{:ATTRIBUTE keyword
 :VALUE     identity
 :CLAUSE    (fn [a v] {:entity '?e :attribute a :value-input v})
 :QUERY     (fn [& c]
              (let [clauses (map #(assoc % :value (symbol (str "?" (gensym)))) c)]
                {:query {:find  ['?e]
                         :in    (into ['$] (map :value clauses))
                         :where (mapv (juxt :entity :attribute :value) clauses)}
                 :args  (into [:db] (map :value-input clauses))}))}
```

output to query map

```clojure

{:query
 {:find  [?e],
  :in    [$ ?G__14055 ?G__14056],
  :where [[?e :name ?G__14055] [?e :gender ?G__14056]]},
 :args [:db "sam" "male"]}
```


How close is this grammar to the one found in some of the most popular frameworks like rails and django?

The only thing difference is that the path params would be linked
references from left to right with the final param being filtered by the query
params. To achieve this the changes to the grammar are modest:

1. We have to distinguish between path and query parameters 
2. Check if the last path parameters has a value or not

grammar:

```text
QUERY = PATHP <"?"> QUERYP*
QUERYP = (ATTRIBUTE <"="> VALUE [<"&">])
PATHP = (SEGMENT [<"/">])+
SEGMENT = (ATTRIBUTE <"/"> VALUE) | ATTRIBUTE
ATTRIBUTE = WORD 
VALUE = WORD | NUMBER
<WORD> = #'[a-zA-Z]+'
<NUMBER> = #'[0-9]+'
```

However in the transformation/emitting stage this leads to some uncomfortable book keeping
because the head and tail of the path params are now special cases. 

Note genbinding is `(symbol (str "?" (gensym)))`


transformations:

```clojure
{:ATTRIBUTE identity
 :VALUE     identity
 :SEGMENT   (fn [& xs] xs)
 :PATHP     (fn [& xs]
              (reduce
                (fn [clauses [attr value]]
                  (let [from        (last clauses)
                        between-ref (genbinding)
                        in-binding  (genbinding)
                        out-ref     (genbinding)]
                    (concat clauses
                      (cond
                        (nil? from)  [{:entity '?a :base attr :attribute (qualify attr "id") :in-value value :in-binding in-binding}]
                        (nil? value) [{:entity (:entity from) :base attr :out-ref out-ref :attribute (qualify (:base from) attr) :ref between-ref}]
                        :else        [{:entity (:entity from) :attribute (qualify (:base from) attr) :ref between-ref}
                                      {:entity between-ref :base attr :attribute (qualify attr "id") :in-binding in-binding :in-value value }]))))
                []
                xs))
 :QUERYP    (fn [attr value]
              {:attribute attr :value value})
 :QUERY     (fn [path & query]
              (let [{:keys [base ref]} (-> path last)
                    clauses            (into path (map

                                                    (fn [{:keys [attribute value]}]
                                                      {:entity     ref
                                                       :attribute  (qualify base attribute)
                                                       :in-binding (genbinding)
                                                       :in-value   value})
                                                    query))
                    args (keep :in-value clauses)]
                {:query {:find  ['?a]
                         :in    (into ['$] (keep :in-binding clauses))
                         :where (map (fn [{:keys [entity attribute in-binding ref]}]
                                       [entity attribute (or in-binding ref)]) clauses)}
                 :args  (into ['db] args)}))}

```


emitting to datomic query map again, nothing surprising here:

```clojure
{:query
 {:find [?a],
  :in [$ ?G__17714 ?G__17713 ?G__17705 ?G__17708],
  :where
  ([?G__17710 :staff/gender ?G__17714]
   [?G__17710 :staff/job ?G__17713]
   [?a :company/id ?G__17705]
   [?a :company/building ?G__17707]
   [?G__17707 :building/id ?G__17708]
   [?G__17707 :building/staff ?G__17710])},
 :args [db "male" "developer" "1" "2"]}

```

We have extended our toy language just a bit to the one of the most common languages of the web.
Used by popular framewords such as Rails, Django, etc..

a detailed set of it's shortcomings would be easy to create by starting with an robust query langauge and
asking how to express it. 



This in no way follows the same organization and grammar rules we just produced. 


## New Perspective

Lets assume i have convinced you we can interpret the URL just like any other programming language. So what? why is that useful?

Lets explore some examples of how this new perspective changes how we view things:

1. Ruby on rails defines a programming language between the database server and client.
2. When the lead developer added a regex to parse the query parameters values on a pipe to mean logical OR he was extending the programming language.
3. When the junior dev asked the team lead how he could avoid the N+1 problem he was asking how to understand the limits of the grammar of programming language. 
4. When Facebook's graphQL is a competing language to the path+queryparam language

The answer is because [discovering](https://youtu.be/IOiZatlZtGU?t=2174) the correct language is hard. It's unlikely that the lessons
in highly mathematical languages have no baring on over the wire communication. It means that likely large amounts of work have been done in this field
and we can tap into that. 

## Progress is more katamari

<img src="/img/katamari.png" width="750px" >

Katamari is a game where you push around a ball of stuff and the bigger it gets the bigger the
things you can pick up. It's a game of momentum. Similarly life is about momentum. The url parameter language
family is here to stay and their are really good solutions in every language for it. In clojure for instance the [reitit](https://github.com/metosin/reitit)
library lets you build the router (the url compiler) using data structures which in turn allows for the router itself to be the output of some function.


```clojure
(require '[reitit.core :as r])

(def router
  (r/router
    [["/api/ping" ::ping]
     ["/api/orders/:id" ::order]]))

(r/match-by-path router "/api/ping")
; #Match{:template "/api/ping"
;        :data {:name ::ping}
;        :result nil
;        :path-params {}
;        :path "/api/ping"}
```

This is about as good as handling the SSP language can get. 

### Alternatives

The clojure community is ripe with alternatives to SSP. Some are tied into frameworks and projects. Here is a quick list of a few i know about:


1. [EQL](https://edn-query-language.org/eql/1.0.0/what-is-eql.html) + [Pathom](https://wilkerlucio.github.io/pathom/v2/pathom/2.2.0/introduction.html)

Tony Kay's brilliantly designed [Fulcro](https://fulcro.fulcrologic.com/) framework uses EQL (a subset of datomics pull query syntax)

As a light intro to EQL, consider you want to following data:

```clojure
{:album/name "Magical Mystery Tour"
 :album/year 1967}
```

So the EQL you pass the server would just be those keys:


```clojure
[:album/name :album/year]
```

Pathom is responsible for parsing your EQL. It's fulfilling the traditional role of a router.

2. [Trident](https://github.com/jacobobryant/trident)

Jacob Bryant's exploration into this topic is worth following. He built a recommendation system 
[Findka](https://findka.com/) and turned the code behind it a set of tools contained
in [trident](https://github.com/jacobobryant/trident). He talks about moving away from fulcro 
[here](https://jacobobryant.com/blog/2020-01-13/)

> The biggest issue I had with Fulcro is that there's a lot of added complexity
> involved with maintaining the client-side database. One of Fulcro's main
> features is automatic normalization: you define your data model as a tree
> structure, inline with your UI, and then Fulcro converts that into a
> normalized graph structure for you. However, in order to make the automatic
> normalization work, you have to do a fair amount of work whenever you read or
> write data.


3. [factUI](https://github.com/arachne-framework/factui)

> FactUI is a library for efficiently rendering user interfaces declaratively, based on a tuple-oriented data store.


4. [Posh](https://github.com/mpdairy/posh)

Model your frontend state using datascript which is based on datomic. 

6. [Odoyle-rules](https://github.com/oakes/odoyle-rules)

Rules engines reactivity update your database based on incoming data in the most efficient way possible. If you store your html state in your database and sync it to your front end, then you achieve `web after tomorrow` like semantics. Further more, your application is more functional because instead of placing functions inside state (the html tree). You put functions in the drivers seat

```clojure
(def components
  (orum/ruleset
    {click-counter
     [:what
      [::global ::clicks clicks]
      :then
      (let [*session (orum/prop)]
        [:button {:on-click (fn [_]
                              (swap! *session
                                (fn [session]
                                  (-> session
                                      (o/insert ::global ::clicks (inc clicks))
                                      o/fire-rules))))}
         (str "Clicked " clicks " " (if (= 1 clicks) "time" "times"))])]}))

```




6. [Hyperfiddle](https://www.hyperfiddle.net/)

Hyperfiddle asks the question what if we decoupled in the right places and pushed composition to the limits. It syncs your frontend to your database by providing a unified language similar to datomic tree pull language with the notable differences

1. where children can reference parents which allows for more powerful relationship joins.
2. any function can join, not just a keyword. Think multimethods instead of protocols. e.g 

Think instead of datomic pull instead a datomic query. you had a datomic query inside your datomic pull.

### Discovering what comes next for "you"

Never ask the question "whats best". It's misleading because it assumes some universal truth. When it comes to
picking a language be it clojure vs java or SSP vs EQL you have to personalize the question. What are the goals?
If you want to pick a language purely based on expressiveness and simplicity then i see no reason why you can't send
a datomic query on the wire. (read Jacobs blog post for details). If your part of a large corporation with little 
time and energy to innovate then you will likely end up with SSP but now your armed with a new perspective which you 
can use to communicate tradeoffs. 

## references

1. https://news.ycombinator.com/item?id=21738331
2. elements of clojure
3. [history of the url](https://blog.cloudflare.com/the-history-of-the-url/)
https://www.moesif.com/blog/technical/api-design/REST-API-Design-Best-Practices-for-Parameters-and-Query-String-Usage/
https://hackernoon.com/restful-api-designing-guidelines-the-best-practices-60e1d954e7c9
https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_is_a_URL
https://www.educative.io/blog/behind-the-screens-what-happens-when-you-type-a-url-in-a-browser
https://restfulapi.net/
