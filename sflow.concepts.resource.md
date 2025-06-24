---
id: xmdevh3s6gvp93m3nyc6683
title: sf-resource
desc: ''
updated: 1750742807245
created: 1750709094321
---

In RDF-land, a resource is any node in an RDF graph that can be represented with an IRI. 

(The other kinds of RDF graph nodes are literals and blank nodes)

In the Semantic Flow, there are several [[kinds of sf-resources|sflow.concepts.resource.type]].

- [[sflow.concepts.resource.namespace]] 
- [[sflow.concepts.resource.thing]]
- [[sflow.concepts.resource.dataset]]
  - [[sflow.concepts.resource.dataset.versioned]]
- [[sflow.concepts.resource.asset-tree]]

```file
Resource                (hasId)  [abstract]
├─ Namespace            (canContain)
├─ Thing                (hasReferent, canContain)
│   └─ Dataset          (hasReferent, hasData, canContain = false)
│       └─ Versioned DS (hasReferent, hasData, isVersioned, canContain = false)
└─ Asset Tree           (isAssetTree)
```