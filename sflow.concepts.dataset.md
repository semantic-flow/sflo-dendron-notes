---
id: 6ki4upsdymjb9vy82oec2cf
title: dataset
desc: ''
updated: 1750710516253
created: 1750655553990
---

In the RDF universe, a dataset is a collection of one or more graphs. 

In the Semantic Flow, a dataset is represented by a [[sflow.concepts.node.dataset]]


## RDF file extensions Support

| Serialization | supported? | Media Type                            | Typical File Extension(s) | Notes                                         |
| ------------- | ------------------------------------- | ------------------------- | --------------------------------------------- |---|
| **JSON-LD**   | in progress | `application/ld+json`                 | `.jsonld`, `.json`        | JSON-based Linked Data.                       |
| **Turtle**    | in progress |`text/turtle`                         | `.ttl`                    | Widely used, human-readable.                  |
| **TriG**      | n | `application/trig`                    | `.trig`                   | Turtle + named graphs.                        |
| **RDF/XML**   | n | `application/rdf+xml`                 | `.rdf`, `.xml`            | The original W3C syntax, verbose XML.         |
| **Notation3** | n | `text/n3` (informal)                  | `.n3`                     | Superset of Turtle with logical implications. |
| **N-Triples** | n | `application/n-triples`               | `.nt`                     | Line-based, one triple per line.              |
| **N-Quads**   | n | `application/n-quads`                 | `.nq`, `.quad`            | Like N-Triples but with graph label.          |
| **HDT**       | n | (binary)                              | `.hdt`                    | Compressed, random-access format.             |
