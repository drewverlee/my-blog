{:title "Singleton Pattern"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2017-5-09"
 :tags  ["Clojure" "Design Pattern" "Java" "Software"]}

<img src="/img/stand_out.jpg">

### Intent

GOF says the intent of The singleton pattern is:

> Ensure a class only has one instance, and provide a global point of access to it.

That sounds like a what rather then a why, which is what we should lead with.

Lets check the motivation: 

>It’s important for some classes to have exactly one instance. Although there can be many printers in a system, there should be only one printer spooler. There should be only one file system and one window manager. A digital filter will have one A/D converter. An accounting system will be dedicated to serving one company.

> How do we ensure that a class has only one instance and that the instance is easily accessible? A global variable makes an object accessible, but it doesn’t keep you from instantiating multiple objects.

This seems rather abstract, why can their only be one printer spooler. Computer programs don't care about physical limitations,
we can have a hundred instances of the printer spool.

Maybe if we check the Consequences

>  Controlled access to sole instance. Because the Singleton class encapsulates its sole instance, it can have strict control over how and when clients access it.

But why is that good thing? I can think of reasons, but i feel the authors should be more explicit. Unfortantly they 
seem content to dance around the issue. I'll be read between the lines then:

*A singleton is global mutable state* 

The authors might quibble at this, but consider:

* global: the singleton is available everywhere
* mutable: if the thing was immutable their would be no reason to protect it
* state: their is no reason to have one instance of a function so that means this is about data/state

### In clojure

We generally avoid generally avoid anything global or mutability. So this rather seems like
an anti-pattern to me. So i would say the context is so important here that it would be unhepful to give
an example. If you need to worry about state then i suggest reading about how  [how clojure handles state](https://clojure.org/about/state)
If your problem is more about tasteful life cycles then take a look at [component](https://github.com/stuartsierra/component).

