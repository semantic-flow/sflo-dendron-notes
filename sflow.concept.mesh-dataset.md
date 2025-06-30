---
id: 6ki4upsdymjb9vy82oec2cf
title: dataset
desc: ''
updated: 1751237370620
created: 1750655553990
---

In the RDF universe, a dataset is a collection of one or more graphs. 

A **semantic flow dataset** (sflow-dataset for short) is represented by a [[sflow.concept.mesh.resource.folder]] that must contain at least one [[distribution file|related-topics.dataset.distribution]]. All distribution files should contain the same data, and must be named the dataset's [[sflow.concept.namespace.segment]]. E.g., a dataset folder named "monsters" could contain distributions named "monsters.jsonld" and "monsters.ttl". 

[[sflow.concept.mesh.resource.node.dataset]]are a kind of semantic flow dataset that have a [[sflow.concept.mesh.resource.element.catalog-dataset]] and may contain other [[mesh nodes|sflow.concept.mesh.resource.node]].

Consistent with [[DCAT v3|related-topics.dcat.vocabulary]], [[sflow.concept.dataset-series]] are also [[sflow.concept.mesh-dataset]] themselves.

Some [[sflow.concept.mesh.resource.element]] are also sflow-datasets: 
- [[sflow.concept.mesh.resource.element.catalog-dataset]]
- [[sflow.concept.mesh.resource.element.reference-dataset]]
- [[sflow.concept.dataset.versioned]]
- [[sflow.concept.mesh.resource.element.v-series-dataset-series]]



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