{:title "RDD with shadow cljs, deps, emacs and cider"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2020-7-19"
 :tags  ["Clojure" "Software" "Emacs" "Spacemacs" "cider" "shadow-cljs" "deps"]}
 

## Motivation 

Repl driven development (RDD) is a powerful tool and can even enhance other feedback mechanisms such as testing.
One of the challenges of RDD is that you need to leave development options up to the developer. This means
that configuration is often spread out on a devs computer and spans multiple files, environment variables
etc. Simple put, There are a lot of options for RDD in clojure and in this blog post just be highlighting one example.

Shadow + clojure deps + emacs + cider

The docs for each of these are great, but their are a lot of options and its easy to get off the happy
path the maintainers have setup. Also because the breadth of options these tools span, its necessary for you to do a bit of configuration 
to get on the happy path. 

## Happy path 

If you follow the initial setup on shadow website you might end up with a shadow-cljs.edn that looks like this:

```clojure
{:source-paths
 ["src/dev"
  "src/main"
  "src/test"]

 :dependencies
 []

 :builds
 {}}
```

To convert this projects to use clojure deps you add the `:deps`  key and point it true or a map with aliases you want to invoke. I find the later is what i end up with. 

```clojure
{:deps     {:aliases [:dev]}
 :dev-http {8080 "public"}

 :nrepl {:middleware [refactor-nrepl.middleware/wrap-refactor]}

 :builds
 {:frontend {:target   :browser
             :devtools {:repl-pprint true
                        :after-load  acme.frontend.app/init
                        }
             :modules  {:main {:init-fn acme.frontend.app/init}}}}}
```

Also note that we configure nrepl to use refactor-nrepl middleware. This is important as other wise you get warnings and it's possible the functionality is broken.

Now shadow defers to deps.edn to manage the dependencies. As the shadow docs point out, we need to list shadow as
a dependencies explicitly if we use deps.edn:

```clojure
{:paths   ["src/main"]
 :aliases {:dev {:extra-deps  { thheller/shadow-cljs {:mvn/version "2.10.15"}}}}}
```

Note we can put shadow as a dev dependency because its not used in our app itself. Only to build and develop it.

Finally we want a way to do RDD on our browser client. This is managed by the popular emacs package cider. via
`cider-jack-in-cljs` for shadow projects. 

But we want to tell emacs some defaults for our project, like that we want to use shadow and 
what shadow app to build and watch. We do this via the dir-locals.el which needs to rest in the root of 
your project. 


```emacs
((clojurescript-mode
  ;; You use a shadow-cljs to build the project
  ;; This answers the question "which command should be used?"
  (cider-preferred-build-tool . shadow-cljs)
  ;; This sets a default repl type and  answers the question "select cljs repl type".
  (cider-default-cljs-repl . shadow)
  ;; This tells shadow cljs what to build and should match a key in your shadow-cljs.edn
  ;; build map. e.g :builds {:<some-key> {...}}
  ;; pramas passed to shadow-cljs to start nrepl via cider-jack-in
  (cider-shadow-default-options . "frontend")))

```

Note that this differs slightly by the one in the shadow docs, which i believe is invoked no matter the project as where by using "clojurescript-mode"
this is tied to running `cider-jack-in-cljs`. Running said command given that dir locals file is displayed in the *messages* in a emacs buffer as well as in the repl:


```
[nREPL] Starting server via /usr/bin/npx shadow-cljs -d nrepl:0.8.0-alpha5 -d cider/piggieback:0.5.0 -d refactor-nrepl:2.5.0 -d cider/cider-nrepl:0.25.3-SNAPSHOT server
[nREPL] server started on 33917
[nREPL] Establishing direct connection to localhost:33917 ...
[nREPL] Direct connection to localhost:33917 established
```

So we inject a number of dependencies cider needs. It's ideal that this injection be managed here and by cider itself because those versions need to 
match the emacs client versions. If there out of sync you will get warnings and you will develop in fear that something is wrong. Never a good experience.

## Help them help you.

Now your development experience is simple open the project and run cider jack in. Great. Given the requirements i think this
is the cleanest version. But the development experience is as programmable as your app itself. Ideally you don't spend your time developing your 
dev environment instead of your app. That's why its crucial to support these projects:

* [shadow](https://github.com/sponsors/thheller)
* [cider](https://github.com/sponsors/bbatsov)
* [spacemacs](https://www.paypal.com/webapps/shoppingcart?flowlogging_id=b5d93605fc3b3&mfid=1595194719571_b5d93605fc3b3#/checkout/openButton)
* [spacemacs help](https://practicalli.github.io/#support)

The help from these maintainers is amazing. I simply don't know how they find the energy to put up with me alone! (i jest, i try to be low maintenance.)

## Recap

cider looks at dir-locals to see that it should use shadow which configures nrepl and defers to deps.edn

This is, in my experience, the most common path you need to be on. But the sky is the limit, 
as an example take a gander at what Plexus is doing here [chui](https://github.com/lambdaisland/chui)

the dir locals picks clojure-cli and then injects the shadow middlware. It further sets up a dev namespace
where you can call the shadow clojure functions to start and stop the server. 
