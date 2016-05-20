---
title: Datascript 101 - Chapter 3
summary: Learn how to bind and destructure data
categories: [clojurescript, clojure]
layout: post
date: 2016-05-20
comments: true
---

We're going to do some pretty nifty stuff, first lets change the data around
a bit, lets do some users instead of cars.


{%highlight clojure%}
(def schema {:user/id {:db.unique :db.unique/identity}
             :user/name {}
             :user/age {}
             :user/parent {:db.valueType :db.type/ref
                           :db.cardinality :db.cardinality/many}})

(def conn (d/create-conn schema))

(d/transact! conn
             [{:user/id "1"
               :user/name "alice"
               :user/age 27}
              {:user/id "2"
               :user/name "bob"
               :user/age 29}
              {:user/id "3"
               :user/name "kim"
               :user/age 2
               :user/parent [[:user/id "1"]
                             [:user/id "2"]]}
              {:user/id "4"
               :user/name "aaron"
               :user/age 61}
              {:user/id "5"
               :user/name "john"
               :user/age 39
               :user/parent [[:user/id "4"]]}
              {:user/id "6"
               :user/name "mark"
               :user/age 34}
              {:user/id "7"
               :user/name "kris"
               :user/age 8
               :user/parent [[:user/id "4"]
                             [:user/id "5"]]}])
{%endhighlight clojure%}

We setup a schema which will help us identify users with a lookup-ref `[:user/id <id>']`, then we have a `:user/parent` attribute which will point to other refs (other users in our case) and will be an array.  We also insert some dummy data.

# Level 3 - Querying

First lets try to pull out the IDs of all the users we know of

{%highlight clojure%}
(d/q '[:find ?e
       :where [?e :user/id]]
     @conn)
;; => #{[1] [2] [3] [4] [5] [6] [7] [8]}
{%endhighlight clojure%}

The first thing to notice here is the data is layed out in a pretty weird format, a set of 1-tuples? what the ...

What if I query two attributes?

{%highlight clojure%}
(d/q '[:find ?e ?n
       :where
       [?e :user/id]
       [?e :user/name ?n]]
     @conn)
;; => #{[1 alice] [2 bob] [3 kim] [4 aaron] [5 john] [6 mark] [7 kris] [8 paul]}
{%endhighlight clojure%}

Ok, nice.  This kind of output is called relations (may be shows you how data is related?).  But from processing point of view it could be weird to do processing on this just to get the data the way you want it.  We can use special syntax to return other things:

{%highlight clojure%}
(d/q '[:find [?e ...]
       :where
       [?e :user/id]]
     @conn)
;; => [1 2 3 4 5 6 7 8]
{%endhighlight clojure%}
That weird `[?e ...]` part says that return me the output as a vector.  Unsurprisingly, you can only query one attribute at a time like this (pull helps solves this, but that's for another day).

{%highlight clojure%}
(d/q '[:find [?n ...]
       :where
       [?e :user/id]
       [?e :user/name ?n]]
     @conn)
;; => [alice bob kim aaron john mark kris paul]
{%endhighlight clojure%}

What if I am expecting on one item to be returned (or can I coerce the output to only return one item?)

{%highlight clojure%}
(d/q '[:find ?n .
        :where
        [?e :user/id]
        [?e :user/name ?n]])
;; => alice
{%endhighlight clojure%}

Here the output was coerced into a "scalar" (single value), doesn't really make much sense here, but sometimes you're expecting to find a single thing and this is a nice way of avoiding an `ffirst`.

These are some of the variations I've found useful.  I think there are more (tuples), but I haven't really encountered them yet, so that'll have to wait.

