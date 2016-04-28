---
title: Datascript 101 - Chapter 1
summary: Learn how to initialize Datascript and Insert some records.
categories: [clojurescript, clojure]
layout: post
comments: true
---

Assuming that you have Datascript referred as `d`, Initialize a database:

{%highlight clojure%}
(def conn (d/create-conn {})
{%endhighlight clojure%}

The `{}` is schema, where you define how certain attributes (think fields) behave: "Are they references?", "Are they arrays?".


{%highlight clojure%}
(def schema {:car/maker {:db/type :db.type/ref
             :car/colors {:db/cardinality :db.cardinality/many}}

(def conn (d/create-conn schema})
{%endhighlight clojure%}

Now we have a database `conn` with some schema, we're saying:

> "I will have a `:car/maker` attribute which will be a reference to some other entity (think record), so do the needful when I ask you deref it. Also, `:car/colors` is going to be an array."

Basically hints to Datascript to make our lives easier.

Now, lets insert some schnitzel:


{%highlight clojure%}
(d/transact! conn [{:maker/name "Honda"
                    :maker/country "Japan"}]
{%endhighlight clojure%}

What's going on here?

 - `transact!` means we're going to transact something (insert/delete/update).
 - `conn` is our database (or connection to if you will).
 - The weird looking array is what we intend to transact.  There are several ways that can be specified as well learn in future lessons.

I didn't add any of the attributes I defined up in my schema, you only need to define stuff that you want to specifically control.  Lets do some more:


{%highlight clojure%}
(d/transact! conn [{:db/id -1
                    :maker/name "BMW"
                    :maker/country "Germany"}
                   {:car/maker -1
                    :car/name "i525"
                    :car/colors ["red" "green" "blue"]}])
{%endhighlight clojure%}

What in the ...?  Well, lets disect it a bit:


You're saying:

> "Insert a maker and a car made by that maker, give the maker an id -1, then add a car and link up the maker for that car with the maker with id -1."


Rememeber we marked `:car/maker` as a reference in our schema, so it will take IDs of other entities (makers).  But we don't know the ID of the maker yet :(,


