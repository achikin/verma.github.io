---
title: Firebase with Clojure
summary: Make firebase more idiomatic with clojure.match
categories: [clojure]
layout: post
---

You love Firebase? I love Firebase.

_WARNING: Fun with transducers ahead.  Note that the use of transducers could have been totally avoided, but what's the point if we're not having a little fun, right? right?_

Firebase has made client side web-apps development trivial for me. I can forget about managing my data, pushing it, sanitizing it (may be a little) and retrieving it.  These things may seem simple but having them taken care for you puts you in a different _State of Mind_ which takes that data management burden out of your thought process.

I've been working on a Firebase Clojure library called [pani](https://github.com/verma/pani) (hindi word for water).  Things are flowing (no pun intended) nicely, although Clojure support is lagging behind ClojureScript (since I tend to use the latter more).

In this post I am going to touch on one aspect of pani. I will use certain functions which I've already coded for pani (I will try to explain them as I use them).  What I intend to do is write a function named `listen<` which listens for events on Firebase `refs` and provides a nice way of dealing with them.  We'd be using `core.async` and `core.match`.

### First things first
The function name `listen<` has that angle bracket in it indicating that this function will be returning a `core.async` channel.  _Events_ will be posted to this channel as clojure maps which we will deconstruct using `core.match` (stay with me if you feel like we're walking into a cloud).

I will refrain from using certain functions from _pani_ itself to make things clearer (although it would have resulted in more concise code).

I am not going to show you the requires and stuff since this function will be a part of the _pani_ library and requires have already [been taken care of](https://github.com/verma/pani/blob/master/src/pani/cljs/core.cljs#L1).


### Basic Structure
I think the function should look something like:

{% highlight clojure %}
(defn listen<
  "Given a Firebase root and a key (or a seq of keys) return a  
   channel which will deliver events"
  [root korks]
  (let [c (chan)]
    c))
{% endhighlight %}

The function takes a Firebase root (can be created with `pani/root` function and either a single key or a seq of keys).  We just declare a `chan` for now and return it.

Let's build on it.

### Listening for Firebase Events
Although _pani_ already has functions to listen for Firebase events, let's just re-write them here using some transducers for extra giggles.

Mostly we're just interested in three Firebase events here: `child_added`, `child_removed` and `child_changed` (Let's collectively decide to not worry about `child_moved`).  Also for most use cases I've found that I never need the previous snapshot or node name (let me know if that's not the case though, or I'll sooner or later hit one).

First, let's write a little function which takes a Firebase ref and returns to us a `chan` which will have the received value posted to it (`pani/bind` does something similar) after its passed through a provided transducer.

{% highlight clojure %}
(defn- fb->chan
  "Given a firebase ref, an event and a transducer, binds and posts to returned channel"
  [fbref event td]
  (let [c (chan 1 td)]
    (.on fbref (clojure.core/name event)
         #(put! c [event %]))
    c))
{% endhighlight %}

This function takes a Firebase ref, an event in the form of a keyword e.g. `:child_added` and a transducer.  The function pushes values as a vector into the channel.  The values look like `[event-type firebase-snapshot]`, e.g. `[:child_added #js {:firebase "stuff"}]` (that `#js` is called a _reader_, basically saying what follows needs to be interpreted as a javascript object, makes more sense when you're reading in code, hence the name).  The transducer then accepts this vector and turns it into a nice looking map, something like `{:child_added ["key name" "value"]`.  Like I said before, we can totally not use a transducer here, but we're just having a little bit of fun.

Testing this is not a trivial thing to do, so you'd have to take my word for it that it's working fine.  If you really want to test it, you can create a new ClojureScript application, refer to _pani_ and play around with it. _Hint: a good starting point for ClojureScript apps is David Nolen's [mies](https://github.com/swannodette/mies)_.

### Using our new function
To use this function we're going to use `clojure.match`. The specific structure (a map) the `listen<` function delivers is to make it easy to deconstruct it using `clojure.match`.  To begin with, let's just print what we receive.

{% highlight clojure %}
(def r (p/root "https://secret-app.firebaseio.com/"))

(let [c (p/listen< r [:items])]
  (go-loop [msg (<! c)]
           (println msg)
           (recur (<! c))))
{% endhighlight %}

We define `r` as the root of our Firebase app.  We call `listen<` on it and pass it `[:items]`, since here we're interested in `/items` ref.  Any items added, removed or changed under this ref needs to be told us about.  When I run this and simulate adding and removing values, I get output like this:

    {:child_added [4 "clojure"]}
    {:child_added [5 "rocks!"]}
    {:child_changed [5 "rocks hard!"]}
    {:child_removed [5 "rocks hard!"]} 

Ok, so far so good.

Let's say I want to keep a track of values under my Firebase ref.  I would start with an empty map and as I receive notifications about items being added, removed or modified under the ref, I want to update my map accordingly.  I would do it something like:
