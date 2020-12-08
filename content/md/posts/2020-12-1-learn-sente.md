{:title "Lets Learn Sente and its friends!"
 :layout :post
 :klipse {:settings {:codemirror-options-out {:line-numbers false}}}
 :date "2020-8-17"
 :tags  ["Clojure" "Software" "Sente" "LetsLearn"]}

<img src="/img/TODO-sente-fullpicture.png">

## Introduction

This will be the first in a series i'll be tagging "LetsLearn". Where i'll 
be explaining the goal and demonstrating how to use a clojure library.

The first library were going to look at is [Sente](https://github.com/ptaoussanis/sente). 
but were also going to bounce around, why? because its really hard to understand clojure libraries 
in isolation. It's easier to dance around the landscape then put on blinders.


## Goal

Depending on your background, there are multiple ways to state the goal. I'm going to state the goal in several ways
and then the rest of this post i'll explain how to understand these goals. Dont worry if you dont understand them now.

* Sente is full duplex communication between a server and client using _todo_

The README states

> Sente is a small client+server library that makes it easy to build reliable, high-performance realtime web applications with Clojure + ClojureScript.

<img src="/img/sente-bf.png">


So it spans both frontend and backend.

You can see this play out in a minimal clojure framework called [biff](https://github.com/jacobobryant/biff/search?q=sente)


> Or: We don't need no Socket.IO

So Sente is does the same thing as Socket.IO. If we consult an [overview](https://www.ably.io/topic/socketio) of Socket.IO. 

We learn socket.IO relies on WebSockets. Which makes sense because Sente does as well.

> Or: core.async + Ajax + WebSockets = The Shiznizzle

it would be accurate to say Sente = core.async + ajax + websockets. But if your not familiar 
with those technologies that wont tell you anything. So lets break them down.

<img src="/img/sente-async-photo.png">

# WebSockets

Websocket => full duplex TCP communication protocol. 
Put another way, websockets allow for two way communication between a server and a browser application. 

This is done by establishing a [websocket handshake](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_servers)


In the context of Sente, we would say sente uses websockets to help you tell your server how to send messages (e.g edn) to your frontend javascript application.

I forked [Sente](TODO) for educational purposes, you can see that it contains Java-WebSockets:

```clojure
{:deps
 {org.clojure/core.async            {:mvn/version "1.3.610"}
  com.taoensso/encore               {:mvn/version "3.1.0"}
  org.java-websocket/Java-WebSocket {:mvn/version "1.5.1"} <=====
  org.clojure/tools.reader          {:mvn/version "1.3.3"}
  com.taoensso/timbre               {:mvn/version "4.10.0"}}}
```


Looking at the dependencies is also a good way do a meta analysis on a library. A library with a lot of unexpected dependencies might require more attention. The only dep i personally dont recognize is [encore](https://github.com/ptaoussanis/encore/blob/master/src/taoensso/encore.cljc) which is the authors personal library of helper methods. Meaning, they extended Clojure with some idioms they find helpful:

For example, they modified `if-let` so it can take multiple bindings, which it achieves by recursively 
calling itself.

```klipse-cljs

(ns foo.core$macros)
(defmacro if-let
  "Like `core/if-let` but can bind multiple values for `then` iff all tests
  are truthy, supports internal unconditional `:let`s."
  {:style/indent 1}
  ([bindings then     ] `(if-let ~bindings ~then nil))
  ([bindings then else]
   (let [s (seq bindings)]
     (if s ; (if-let [] true false) => true
       (let [[b1 b2 & bnext] s]
         (if (= b1 :let)
           `(let      ~b2  (if-let ~(vec bnext) ~then ~else))
           `(let [b2# ~b2]
              (if b2#
                (let [~b1 b2#]
                  (if-let ~(vec bnext) ~then ~else))
                ~else))))
       then))))
       
(foo.core/if-let [x 2 y 2] (+ x y) "fail")
```

But back to websockets! It doesn't look sente is going to make us think in websockets directly, i'm guessing will be using core async for that.
Put another way, core async will abstract _over_ websockets. Will get a bit into core async in a moment. Take a moment and consider websockets from a
security perspective. Knowing how to break something will give you an better understanding of what it is. The key takeaway from the literature is that the 
websocket protocol doesn't check the origin header in the clients handshake request. 

To rephrase that, and give more context, it means that when the server _served_ up the webapplication it set a policy that this webapplication would only 
talk to it. the _same origin policy_, this is controlled at layer outside the applications control, by the browser. 

<img src="/img/sente-same-origin.png">

Often times your security model needs to work around that anyway. e.g your
client has to talk to two domains. Well we don't have that, so what do we have
for security? Well Sente doesn't need to care, but the author left us a hint:


```
    [ring.middleware.anti-forgery :refer [wrap-anti-forgery]] ; <--- Recommended
```

Middleware is called as such because its sits in the middle of the java servlet api which handles the HTTP protocol 
and things which _handle_ GET and POST requets:

<img src="/img/sente-handler.png">

This is useful because it lets the clojure community share common HTTP functions that effect incoming responses 
and outgoing requests. As were about to see. If we jump to that [library](https://github.com/ring-clojure/ring-anti-forgery) then we see 

> Ring middleware that prevents CSRF attacks. By default this uses the synchronizer token pattern.

The premise here is that the server gives the client a key and its verified in every request.

<img src="/img/sente-sync-pattern-TODO.png">

And would look like

```html
<input name ="csrf" value="aldkjflakjdbwoijpokajsdf" </>
```


so if we look at this library we should expect to see it generating a random unique sequence it can check.

we see tests for creating an [input field](https://github.com/ring-clojure/ring-anti-forgery/blob/e248f9f5b0cb593bae998ed3fa0d538f6f651ad0/test/ring/util/test/anti_forgery.clj#L6)

```
(deftest anti-forgery-field-test
  (binding [*anti-forgery-token* "abc"]
    (is (= (anti-forgery-field)
           (str "<input id=\"__anti-forgery-token\" name=\"__anti-forgery-token\""
                " type=\"hidden\" value=\"abc\" />")))))
```

but will want to be sending this as part of our handshake.
From the readme

> The middleware also looks for the token in the X-CSRF-Token and X-XSRF-Token header fields, which are commonly used in AJAX requests.




and we see also see the `synchronizer pattern code`

```clojure
(deftype SessionStrategy []
  strategy/Strategy
  (get-token [this request]
    (or (session-token request)
        (random/base64 60)))

  (valid-token? [_ request token]
    (when-let [stored-token (session-token request)]
      (crypto/eq? token stored-token)))

  (write-token [this request response token]
    (let [old-token (session-token request)]
      (if (= old-token token)
        response
        (-> response
            (assoc :session (:session response (:session request)))
            (assoc-in
              [:session :ring.middleware.anti-forgery/anti-forgery-token]
              token))))))
```

Lets do a quick check to see how unique and random it looks

```clojure
(crypto.random/base64 60)
;; => "Xh2GxMJJw4gMAovSLZRCSUcyKJuTVwcbx24GYI2YWD35TGBLHCpn7GBCDzw6K0B86pxUKbict90PiCBQ"

;; would check uniqueness
(let [n 10000]
  (= n (count (set (take n (repeatedly #(crypto.random/base64 60) ))))))
;; => true
```





## Comparing

The docs are nice enough to point out two alternatives

