# Mundaneum

This is a tiny, highly incomplete clojure wrapper around
the [Wikidata](https://www.wikidata.org/wiki/Wikidata:Main_Page)
project's massive semantic database. It's named after
the [Mundaneum](https://en.wikipedia.org/wiki/Mundaneum), which
was [Paul Otley](https://en.wikipedia.org/wiki/Paul_Otlet)'s mad and
wonderful c. 1910 vision for something like the World Wide Web.

(There's a mini-doc about him and it
[here](https://www.youtube.com/watch?v=hSyfZkVgasI).)

## Motivation

Wikidata is amazing! And it provides API access to all the knowledge
it has collected! This is great, but exploratory programmatic access
to that data can be fairly painful.

The official Wikidata API Java library offers a document-oriented
interface that makes it hard to ask interesting questions. A better
way to do most things is with the Wikidata query service, which uses
the
standard [Semantic Web](https://en.wikipedia.org/wiki/Semantic_Web)
query language, [SPARQL](https://en.wikipedia.org/wiki/SPARQL).

The SPARQL query service is nice, but because the WikiData data model
must cope with (a) items with multiple names in multiple languages,
and (b) single names that map to multiple items, they've used a layer
of abstraction by which everything in the DB is referred to by an `id`
that looks like `P50` (property number 50, meaning "author") or
`Q6882` (entity number 6882, the author "James Joyce").

For example, to get a selection of works authored by James Joyce,
one would issue a query like:

``` sparql
SELECT ?work
WHERE { ?work wdt:P50 wd:Q6882. } 
LIMIT 10
```

(Users of [Datomic](http://www.datomic.com) will recognize the `?work`
style of selector, which is not a coincidence as SPARQL and Datomic
were both strongly influenced
by [Datalog](https://en.wikipedia.org/wiki/Datalog).)

The above query is simple enough, except for the non-human readable
identifiers in the `WHERE` clause, which were both found by manually
searching the web interface at Wikidata.

The first order of business was to build a more human-friendly way to
specify relationships and entities without leaving my coding
environment. The approach I took was:

* download and reformat the full list of ~3000 properties (fresh as of
  2016-12-01), shape them into a map of keyword/string pairs where the
  keyword is the name of the property and the string is its `id`, and
  make a helper function 

``` clojure
(prop :author)
;;=> "P50"
```

* create a helper function that tries to correctly guess the id of an
  entity based on a string that's similar to its "label" (common name,
  currently sadly restricted to English in this code)

``` clojure
(entity "James Joyce")
;;=> "Q6882"

;; also supports additional criteria to help find the right entity:
(entity "U1")
;;=> "Q2472052", which is a Wikipedia disambiguation page

(entity "U1" :part-of "Berlin U-Bahn")
;;=> "Q99691", which is U-1 line that runs north of my flat
```

This already helps to keep my emacs-driven process running
smoothly. The next point of irritation was assembling query strings by
hand, like an animal. So I banged together a quick and sloppy DSL
similar to the one offered by Datomic. This looks like:

``` clojure
;; what are some works authored by James Joyce?
(query '[:select ?work ?workLabel
         :where [[?work (prop :author) (entity "James Joyce")]]
         :limit 10])
;; #{{:work "Q864141", :workLabel "Eveline"}
;;   {:work "Q861185", :workLabel "A Little Cloud"}
;;   {:work "Q459592", :workLabel "Dubliners"}
;;   {:work "Q682681", :workLabel "Giacomo Joyce"}
;;   {:work "Q764318", :workLabel "Two Gallants"}
;;   {:work "Q429967", :workLabel "Chamber Music"}
;;   {:work "Q465360", :workLabel "A Portrait of the Artist as a Young Man"}
;;   {:work "Q6511", :workLabel "Ulysses"}
;;   {:work "Q866956", :workLabel "An Encounter"}
;;   {:work "Q6507", :workLabel "Finnegans Wake"}} 
```

This is actually quite similar to the programmatic query interface I
created for the first
purpose-built [TripleStore](https://en.wikipedia.org/wiki/Triplestore)
around 15 years ago.

## Condition

The state of this code is appalling. It's around six person-hours old,
and currently a flaming mess of fragility and typos. It is presented
for entertainment purposes only.

This project is just for fun, so I'm not really sure if I'll turn it
into a proper library with things like doc-strings and tests and some
possibility of actually working for anyone but me.

The `mundaneum.core` namespace is all examples. Enjoy!

## License

Copyright © 2016 Jack Rusher

Distributed under the Eclipse Public License either version 1.0 or (at
your option) any later version.