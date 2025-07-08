---
id: rregmt56znauz71qgypet6a
title: node component
desc: ''
updated: 1751953040770
created: 1751688486456
---

There are three types of node components, and they are the most important parts of a node: 

**Node components** are dcat:Catalog but their sense is of an abstract dataset that exists through time, independent of any specific version or realization. In a sense, they are "data cakes" made up of  

## Relationship to Node Datasets

As abstract datasets, node components are realized through [[sflo.concept.mesh.resource.element.node-component.layer]], which are specific temporal snapshots or versions of the abstract concept. To borrow a phrase from the PROV model, we say that a layer is a specialization of the abstract dataset.

### Relationship pattern:

Every node component has at least two concrete layers: [[sflo.concept.mesh.resource.element.node-component.layer.current]] and [[sflo.concept.mesh.resource.element.node-component.layer.next]].

The abstract dataset, if versioned, is a [[related-topics.dcat.dataset-series]] and may have multiple concrete datasets, i.e., [[sflo.concept.mesh.resource.element.node-component.layer.version]]s.

- Abstract dataset: "My ontology definitions" (persistent concept)
- Concrete datasets: v1.0, v1.1, current version, working draft of next version (specific realizations)

### Ontology Example

```file
/my-ontology/
├── _ref/                    ← Abstract dataset (reference data about ontology)
│   ├── _current/           ← Concrete dataset
│   ├── _v1/                ← Concrete dataset  
│   └── _v2/                ← Concrete dataset
└── _data/                  ← Abstract dataset (ontology definitions)
    ├── _current/           ← Concrete dataset
    └── _v1/                ← Concrete dataset
```

In this example:

_ref/ represents the abstract dataset of reference information about the ontology
_data/ represents the abstract dataset of the ontology's definitional content
Each _current/, _v1/, etc. contains concrete dataset realizations

## Persistent Identity

Abstract datasets provide conceptual continuity by:

- Maintaining meaning across versions and changes
- Preserving references from external sources
- Enabling evolution while keeping identity stable
- Supporting versioning without losing conceptual coherence
