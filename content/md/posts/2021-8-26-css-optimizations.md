{:title "CSS: Optimizations and Organization"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :tags  ["Clojure" "CSS" "Software" "Web"]}

This post is about css with companion sections from a clojure perspective. 

The high level goal of CSS is to produce a visually appealing website. Humans appreciate the design, but it's machines that set the rules so it
pays to know the constraints and contracts. And as part of what makes something beautify is it's responsiveness and that means talking about optimizations.

## [General rules](https://developer.mozilla.org/en-US/docs/Learn/Performance/CSS)

Here are some rough categories that the over all discussion can be broken into:

* Minify/compressing
* Remove unnecessary styles
* Split css not required at page load into additional files to reduce css render blocking 
* loading
* Critical CSS

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

As explained [here](https://developer.mozilla.org/en-US/docs/Learn/Performance/CSS#optimizing_for_render_blocking) one way to do this is via media queries. This means the styles are still downloaded at the load, but aren't blocking. Another opportunity to do this is through [code splitting](https://shadow-cljs.github.io/docs/UsersGuide.html#CodeSplitting) with reacts lazy load option which you can read about [here](https://code.thheller.com/blog/shadow-cljs/2019/03/03/code-splitting-clojurescript.html). While this isn't specifically aimed at CSS sense your components might have local CSS in them, it's worth mentioning here.

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


# Conclusions

* Minifiction:               A machine should do the work. 
* Remove unnecessary styles: Prevent through sensible clojure coding practices
* load what you need:        Really this is an extension of removing un-necessary styles, achievable with media queries and likely CLJS.
* Loading:                   Have a business plan around the tradeoffs, default to fresh content every time as to not confuse your users. Leverage immutability.
* Inline Styles              Defiantly a win for critical CSS, but likely a time investment.

