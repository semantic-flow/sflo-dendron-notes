---
id: 6ki4upsdymjb9vy82oec2cf
title: dataset
desc: ''
updated: 1751565550579
created: 1750655553990
---

In the RDF universe, a dataset is a collection of one or more graphs. 

A **mesh dataset**  is represented by a [[sflo.concept.mesh.resource.folder.dataset]] that must contain at least one [[distribution file|related-topics.dataset.distribution]]. All distribution files should contain the same data, and must be named the dataset's [[sflo.concept.namespace.segment]]. E.g., a dataset folder named "monsters" could contain distributions named "monsters.jsonld" and "monsters.ttl". 

[[sflo.concept.mesh.resource.node.data]] refer to a dataset that have a [[sflo.concept.mesh.resource.element.meta-dataset]] and may contain other [[mesh nodes|sflo.concept.mesh.resource.node]].

Consistent with [[DCAT v3|related-topics.dcat.vocabulary]], [[sflo.concept.mesh.resource.node.data.series]] are also [[sflo.concept.mesh.resource-facet.dataset]] themselves.

Some [[sflo.concept.mesh.resource.element]] are also mesh datasets: 
- [[sflo.concept.mesh.resource.element.meta-dataset]]
- [[sflo.concept.mesh.resource.element.reference-dataset]]
- [[sflo.concept.mesh.dataset.versioned]]
- [[sflo.concept.mesh.resource-facet.dataset.v-series]]

## Kinds of Mesh Datasets

- [[sflo.concept.mesh.resource.node.data]]
- [[sflo.concept.mesh.resource.element]]

## RDF file extensions Support

| Serialization | supported?  | Media Type              | Typical File Extension(s) | Notes                                         |
| ------------- | ----------- | ----------------------- | ------------------------- |
| **JSON-LD**   | in progress | `application/ld+json`   | `.jsonld`, `.json`        | JSON-based Linked Data.                       |
| **Turtle**    | in progress | `text/turtle`           | `.ttl`                    | Widely used, human-readable.                  |
| **TriG**      | n           | `application/trig`      | `.trig`                   | Turtle + named graphs.                        |
| **RDF/XML**   | n           | `application/rdf+xml`   | `.rdf`, `.xml`            | The original W3C syntax, verbose XML.         |
| **Notation3** | n           | `text/n3` (informal)    | `.n3`                     | Superset of Turtle with logical implications. |
| **N-Triples** | n           | `application/n-triples` | `.nt`                     | Line-based, one triple per line.              |
| **N-Quads**   | n           | `application/n-quads`   | `.nq`, `.quad`            | Like N-Triples but with graph label.          |
| **HDT**       | n           | (binary)                | `.hdt`                    | Compressed, random-access format.             |


## Limitations

- although it can contain its own Asset Tree