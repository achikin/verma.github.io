---
title: Datascript 101 - Chapter 2
summary: Learn how to setup uniqueness and identity.
categories: [clojurescript, clojure]
layout: post
date: 2016-04-28 18:00:00
comments: true
---

Let do this.

{%highlight clojure%}
(def schema {:car/model {:db/unique :db.unique/identity}
             :car/maker {:db/type :db.type/ref}
             :car/colors {:db/cardinality :db.cardinality/many}}
{%endhighlight clojure%}

See that `:car/model` definition.  You're saying:

> "Hey this :car/model field is going to be unique and will be used to identify this entity."

You're not limited to one identity, have as many as you want!

{%highlight clojure%}
(def schema {:car/model {:db/unique :db.unique/identity}
             :car/model-id {:db/unique :db.unique/identity}

             :car/maker {:db/type :db.type/ref}
             :car/colors {:db/cardinality :db.cardinality/many}}

(def conn (d/create-conn schema})
{%endhighlight clojure%}

Now lets insert some stuffs.

## Level 3 - Insertion

{%highlight clojure%}
(d/transact! conn [{:car/model "2016 Accord"
                    :car/model-id "ACC2016"
                    :car/name "Accord"}])
{%endhighlight clojure%}

We inserted an entity with two identity fields.  But what does this do?

{%highlight clojure%}
(d/entity  @conn [:car/model "2016 Accord"])
;; => {:db/id 1}

(d/entity  @conn [:car/model-id "ACC2016"])
;; => {:db/id 1}

{%endhighlight clojure%}

In our last chapter, we had to write a pretty elaborate query to get the entity ID which we could pass to `d/entity`.  Now we can use this little "matcher" query to fetch the same thing for us.  This works because we marked the two fields with `db.unique/identity`.

Note that, `d/entity` returns a "lazy" entity.  When you fetch a certain attribute out of it, it will be resolved for you.

{%highlight clojure%}
(:car/name (d/entity  @conn [:car/model-id "ACC2016"]))  ;; => "Accord"
{%endhighlight clojure%}

## Level 4 - Insertion

Lets try this:
{%highlight clojure%}
(d/transact! conn [{:car/model "2016 Accord"
                    :car/name "AccordSoFancy"
                    :car/colors ["red" "green"]}])

(d/entity @conn [:car/model "2016 Accord"]) ;; => {:db/id 1}

(:car/name (d/entity @conn [:car/model "2016 Accord"]) ;; => "AccordSoFancy"
(:car/colors (d/entity @conn [:car/model "2016 Accord"]) ;; => {:db/id 1}) ;; => "AccordSoFancy"

{%endhighlight clojure%}


