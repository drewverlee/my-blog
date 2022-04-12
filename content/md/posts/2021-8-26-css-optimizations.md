{:title "CSS: Optimizations and Organization"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "CSS" "Software" "Web"]}

This post is about css with companion sections from a clojure perspective.

The high level goal of CSS is to produce a visually appealing website. Humans appreciate the design, but it's machines that set the rules so it
pays to know the constraints and contracts. And as part of what makes something beautify is it's responsiveness and that means talking about optimizations.

## Optimizations

## [General rules](https://developer.mozilla.org/en-US/docs/Learn/Performance/CSS)

Here are some rough categories that the over all discussion can be broken into:

* Minify/compressing
* Remove unnecessary styles
* Split css not required at page load into additional files to reduce css render blocking
* loading
* Critical CSS
* Inline styles vs CSS

However it's important to note that the high level concepts are simply that you want to minimize space (package size) and time (latency). But doing so never comes for free, typically costing either more human time or resources to achieve.

## Minify/compressing

Minifing css is relatively straight forward. From [google developers](https://developers.google.com/speed/docs/insights/MinifyResources):

> Minification refers to the process of removing unnecessary or redundant data without affecting how the resource is processed by the browser - e.g. code comments and formatting, removing unused code, using shorter variable and function names, and so on.


As an example we go from this human readable file:

```css
html,
body {
  height: 100%;
}
```

to this minimal file:

```css
body,html{height:100%}
```


A tool should handle this for you. In the clojurescript world, say if your using shadow-cljs and your styles are inlined then this will happen as part of the minification of the application. If your using a css stylesheet, then you will need to use one of the various libraries out there. Such as [asset-minifier](https://github.com/yogthos/asset-minifier) or postCSS. You can read more about minification [here](https://blog.logrocket.com/the-complete-best-practices-for-minifying-css/).

## Remove unnecessary styles

This is very similar to dead code elimination, which in turn is related to the Halting Problem. Long story short, it's [impossible to do perfectly](https://stackoverflow.com/questions/33266420/why-cant-dead-code-detection-be-fully-solved-by-a-compiler). As this [experience report highlights](https://css-tricks.com/how-do-you-remove-unused-css-from-a-site/) trying to do it manually can troublesome. The lesson here is that you need to prefer prevention over clean up.

How? The same techniques we practice in software development apply. Logic should be isolated, minimal and composed. Consider this example:

```clojure
(defn some-componet
   [:p {:style {:color "red"}}])

```

This style is part of the  component, that means it will only render if the component renders. That's a good coupling because it's direct and private. No chance layers of indirection will cause un-needed styles will end up there. _prefer ridged code_, use indirection only when necessary. It also means the [Google Closure Compiler](https://clojurescript.org/about/closure) has a chance to do dead code analysis.

But the story of this example doesn't stop here, we should note that were using what looks like to be an inline style. You might have a knee jerk reaction that this isn't good. But why? It's not an organizational issue, that hashmap is just clojure. It could be a variable. The historical reason for the pushback comes form another way to optimize which will cover in the cacheing section.

## load what you need.

As explained [here](https://developer.mozilla.org/en-US/docs/Learn/Performance/CSS#optimizing_for_render_blocking) one way to do this is via media queries. This means the styles are still downloaded, but aren't blocking. Another opportunity to do this is through [code splitting](https://shadow-cljs.github.io/docs/UsersGuide.html#CodeSplitting) with reacts lazy load option which you can read about [here](https://code.thheller.com/blog/shadow-cljs/2019/03/03/code-splitting-clojurescript.html). While this isn't specifically aimed at CSS sense your components might have local CSS in them, it's worth mentioning here.

## loading assets

Near last by far from least, is the topic of loading assets. Let's set the premise, this is fundamentally a business decision. e.g how important is the first load? Is it better to have fast content vs up to date content? As an engineer it's your job to bring these options to your teams attention in a easy to understand simplified form. With that in mind, let's talk about the trade offs at a high level and then talk about what knobs we have to turn.

The highlevel trade off is space vs time. What I mean by this is that if you send the user all the assets at once, it will take more time on initial load, but less time on subsequent actions on his part. A business much evaluate this choice, but typically it's good to minimize your first load. Another example would be caching. You could tell the browser to never validate if an asset was expired, this means there wont be a latency penalty to check.

Concerning Caching, I agree with the logic [here](https://web.dev/love-your-cache/) that a sensible default is _no cacheing_. The goal is to make sure content is always fresh and to do that as quickly as possible. This relies on CDN's and telling your browser to always revalidate. If you can have a hot CDN cache near your user base, then _i believe_ they can benefit from each other and share pages. You configure this functionality through the header:

```
Cache-Control: max-age=0,must-revalidate,public
;; or
Cache-Control: no-cache
```

However realize that this works best for moderately fast connections. Another option is to have _immutable_ assets and to save them forever via:

```
Cache-Control: max-age=31536000,immutable
```

In order to make them immutable there are a couple options, but essential the files have to have unique names. While Shadow-cljs has built in support for hashing js modules and code splitting (which you should be doing!). It doesn't seem to give the same treatment to css style sheets, again this isn't an issue if everything is inlined.. Going this route means that any asset update will trigger a new re-fetch as where old assets will be cached by the browse. I believe this could be coupled with a load balancer redirect that always grabs the newest home/index.html and avoiding having to hash your index.

Those are some general strategies, but what about the actual controls for browser cacheing? Those are well covered [here](https://web.dev/http-cache/) and [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching) but come down to several controls on the request and response headers:

* Cache-Control. The server can return a Cache-Control directive to specify how, and for how long, the browser and other intermediate caches should cache the individual response.
* ETag. When the browser finds an expired cached response, it can send a small token (usually a hash of the file's contents) to the server to check if the file has changed. If the server returns the same token, then the file is the same, and there's no need to re-download it.
* Last-Modified. This header serves the same purpose as ETag, but uses a time-based strategy to determine if a resource has changed, as opposed to the content-based strategy of ETag.

Practically speaking, the following Cache-Control configurations are a good start:

* Cache-Control: no-cache for resources that should be revalidated with the server before every use.
* Cache-Control: no-store for resources that should never be cached.
* Cache-Control: max-age=31536000 for versioned resources.

It's recommended to use the Etag over the last-modified because its more accurate. But how do we decide what should and shouldn't be cached? This SO question has a [good answer](https://stackoverflow.com/questions/3492319/private-vs-public-in-cache-control).

Overall, the story about cacheing and loading doesn't change in clojurescript land.

## Critical CSS

This process, of inline styles is most important for the first page load, what some call ["above the fold"](https://web.dev/extract-critical-css/). The css in that first page load is refereed to a "critical", because users typically will leave a site that loads to flow especially at the start. Introductions are important. From Google:

> For best performance, you may want to consider inlining the critical CSS directly into the HTML document. This eliminates additional roundtrips in the critical path and if done correctly can deliver a "one roundtrip" critical path length where only the HTML is a blocking resource.

Luckily there are tools to help do this such as discussed [here](https://web.dev/codelab-extract-and-inline-critical-css/). However i Have no doubt that this is a bit error prone. Additionally, next time you turn around there might be a way the browser can detect this with no intervention and your efforts have been wasted.

## Inline Styles vs CSS

Please read this [post](https://simonadcock.com/are-inline-styles-faster-than-atomic-css/) that concludes that css classes are generally faster then inline styles mostly because of browser internals. That that difference is significant (30%) between the two, but probably negligible for the overall experience (~10ms).

This might be why the react [docs](https://reactjs.org/docs/faq-styling.html#can-i-use-inline-styles) have this to say: have this to day:

> CSS classes are generally better for performance than inline styles.


Here are some assorted readings that in aggregate were less helpful then the post above:

* [are inline styles bad?](https://stackoverflow.com/questions/70051589/are-inline-styles-bad)
* [why do the react docs say inline styles are slow?](https://www.reddit.com/r/reactjs/comments/rercdd/why_do_the_react_docs_say_inline_styles_are/)
* [which has a faster paint time...](https://www.reddit.com/r/css/comments/qf9gld/which_has_a_faster_time_to_paint_inline_styles_or/)

## Conclusions

* Minifiction:               A machine should do the work.
* Remove unnecessary styles: Prevent through sensible clojure coding practices
* load what you need:        Really this is an extension of removing un-necessary styles, achievable with media queries and likely CLJS.
* Loading:                   Have a business plan around the tradeoffs, default to fresh content every time as to not confuse your users. Leverage immutability.
* Inline Styles              Defiantly a win for critical CSS because you don't have to wait for the css sheet to load. BUT I believe there are ways to now load a style sheet at the same time as the HTML. So YMMV.
* Inline Styles vs CSS:      CSS classes seem to be slightly faster according to one mans bench-marking efforts.


# Organization

What I mean by organization of css is akin to "code quality" or "simple design". Which means, it's nearly impossible to talk about it in general! What to do then? Give up? Pick up a cargo cult methodology or framework that deals with orthogonal concerns and pretend it solves the issue? No. The hidden secret to architecture is to only setup structure where needed, motivate correctly, and then trust.

When we discuss organizing css we can think about it from two perspectives.

1. what the browser requires
2. what the business wants

## What the browser requires

Let's first cover some basics of how styles are applied. Go ahead and read [this](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Cascade_and_inheritance) on CSS Cascade and Inheritance. I think the only really interesting functionality here is inheritance. Luckily, it's an old topic. Inheritance is just a tree, the parent functionality flows to the children. That's useful. But it doesn't cover all cases. Sometimes two children with different parents have to share some functionality. That's why tree's are a subset of graphs and why being able to share styles outside Inheritance is useful and necessary.

The next big topic concerning the browser is Layout. Unfortunately, this topic (like all browser concerns) is constantly evolving as the browser adds new features. And you absolutely want to keep up to date with them as they will make doing your job easier, even if it means spending more time learning. Embrace the chaos! At the time of this writing you should be moving towards or using [CSS Grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout). I suggest watching and read everything written by [Jen Simmons](https://www.youtube.com/channel/UC7TizprGknbDalbHplROtag).

## What the business wants

Given we understand the browser, it's api, and it's performance tradeoffs, how do we go about effectively running a business through the lens of css? That's a loaded question, the answer is obviously that it depends!

There are guiding principles though. Do you want team members to be generalists or specialists? If you pick generalists you should prepared to get less then perfect CSS, after all, they might decide that fiddling with that button is far less important then reducing the backed pipeline. Maybe you hope to have both by having architects and specialists? But what are the specialists going to work on when they have run out of things to work on effectively?

Let's move past the general uncertainty and focus on some area's I can speak on with some personal experience. Web development using Clojure. In this case, one organization choice that you could face is using stylesheets vs css-in-js. Luckily, the real choice here is based on how you want to structure your team. If you have dedicated designers, then stylesheets are likely a good separation of concern. But It might not exactly be stylesheets that your designers are using, but collaborative interface design tools like [Figma](https://www.figma.com/) which are at a higher level. However if you have chosen to higher mostly developers and tasked with with design and implementation , then you should invest in a tool like [storybook](https://storybook.js.org/) or [workspaces](https://github.com/nubank/workspaces). Assuming your using react and/or clojurescript.

Earlier I said the real difference between CSS stylesheets and css-in-js was in team dynamics, that's because solutions like [cljss](https://github.com/clj-commons/cljss) allow you structure your html and css tries together using clojure:

```clojure
(defkeyframes spin [from to]
  {:from {:transform (str "rotate(" from "deg)")
   :to   {:transform (str "rotate(" to "deg)")}})

[:div {:style {:animation (str (spin 0 180) " 500ms ease infinite")}}]
```

And then they produce stylesheets! There is just one [hiccup](https://github.com/clj-commons/cljss#composing-styles).

> Because CSS is generated at compile-time it's not possible to compose styles as data, as you would normally do it in Clojure. At run-time, in ClojureScript, you'll get functions that inject generated CSS and give back a class name. Hence composition is possibly by combining together those class names.

This happens because were generating class names/strings

```clojure
(defstyles margin-y {:margin-top 1}) ;; => "some-class"
(defstyles margin-x {:margin-bottom 2}) ;; => "some-other-class"


```

Now we have to do string concatenation of "some-class" and "some-other-class" if we want both. I think it would be a major improvement to overcome this limitation where ever possible (even if it meant using inline styles in some cases!).

The other problem is that by putting our CSS behind marcos (necessary to generate stylesheets). We no longer can no longer user cljs to compose style at runtime. I suspect you can work around this in most cases, but it will probably be something of a creative exercise.
And the reason were doing this is because of performance that might not matter! So then, my tentative recommendation is to start with inline-styles, using a cljs-in-jss like tool. Then benchmark and adapt as needed. Luckily Reagent ships with everything you need. You can just use the style map:

```clojure
(defn some-componet
   [:p {:style {:color "red"}} "hi"])
```

And what about things you can't inline? Well just use a style tag

```clojure
(defn some-componet
   [:div
     [:style "@media screen..."]
     [:p {:style {:color "red"}} "hi"]])
```

At this point you might be a bit shell shocked, that after all this i'm recommending inline styles. But you shouldn't be and i'm not really. I'm recommending being direct and isolated as possible, because indirection can lead to confusion. I'm stressing the "as possible" because obviously if two components need to share styles then you will need to extract them out. How? Like you do any data in clojure.


```clojure

(def styles {:style {:color "red"}} )

(defn some-componet
   [:p styles "hi"])

(defn another-componet
   [:p styles  "hi"])
```

If your a Clojure developer by trade this is great, because i don't have to use CSS less organize and compose my styles. This means you can use clojure core to compose styles. Or you can use a [rules engine](https://github.com/oakes/odoyle-rules) to trigger your html and css based on business logic. Or a proper [graph database](https://github.com/tonsky/datascript). Or what ever you think works best for your situation/business.
When I started this section I said the secret to good architecture was trust and I meant it. I'm telling you that your free to face the true horrors of your business unburnded by the need to learn the next 100 css fads...

The End!

## Bonus section: names are important.

We all agree names are important. They help us communicate intent. That's why I'm struggling to understand why anyone would trade "font size: smaller" for "Fz-s". But that's exactly the kind of thing you will see bundled into ideas like tailwind and atomic design:

```css
.Fz-s {
    font-size: smaller;
}
```

Further more, this is munging the key and value together which limits composability options at this level.

From a business perspective, your now going to pay more because you need people be fluent in names used by Tailwind, Atomic design, BEM etc... instead of the much larger pool of people that know CSS. Why? those aren't style guides. Those are tools to help you overcome css limitations, limitations which are constantly changing. Limitations many of which we can use cljs for! The concerns these frameworks are really helping with are completly orthogonal to obscuring names. You can and should avoid doing this.
