{:title "Interpreter Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "Design Pattern" "Java" "Software" "Behavrioral"]}


### Intent

> Given a language, define a represention for its grammar along with an interpreter that uses the representation to interpret sentences in the language.

So this is a pattern for writing a langauge, which might be the same thing as writing a DSL
depending on what your trying to do. The first thing to consider is if 
you want an [external or internal DSL](https://martinfowler.com/bliki/DomainSpecificLanguage.html).


### In clojure

If your building an internal DSL, then your in luck. CLojure is a Lisp and you can make a DSL like so:

```klipse-cljs
;; ignore the namespace
(ns my.m$macros)

(defmacro backwards
  [form]
  (reverse form))
```

```klipse-cljs
(my.m/backwards (" backwards" " am" "I" str))
```

This macro `backwards` intercepted the form `(" backwards" " am" "I" str)` and reversed it
to  `(str "I" "am " "backwards")` before that form evaluated. This is important, because
The operator must be the first thing in the form/list. See what happens without the macro?

```klipse-cljs
(" backwards" " am" "I" str)
```

This is made possible because Clojure is a Lisp and so you as a programer can manipulate both
the regular form (what any language can do) but also the form created the parser and lexer:

<img src="/img/lisp-eval.png">



### Summery

Generally speaking, you should *strongly* prefer functions over macros. 
However their are times when using a macro can greatly simplify a problem. Here is a [blog
post](http://www.lispcast.com/when-to-use-a-macro) about when to use macros if
you want to learn more. But the three main situations are:

* gaining control over the control structure (we demoed this above)
* Performing compile-time computation
* Providing custom syntax
