---
id: 0n1lq6aq1gskj46bpcx9h4h
title: layer distribution
desc: ''
updated: 1751992410510
created: 1751138751433
---

- [[sflo.concept.mesh.resource.element.node-component.layer]] should have one or more distributions.
- a layer's distributions should contain the same data, just in different syntaxes 

## Naming

- [[sflo.concept.mesh.resource.element.node-component.metadata]] and [[sflo.concept.mesh.resource.element.node-component.reference]] have their distributions named with "_meta" and "_ref" respectively, but
- [[sflo.concept.mesh.resource.element.node-component.data]], being the "payload data", 


## Issues

- TODO: should we only support "named graph"-support formats

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
