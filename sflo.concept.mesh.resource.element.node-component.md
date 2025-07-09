---
id: rregmt56znauz71qgypet6a
title: node component
desc: ''
updated: 1752025335642
created: 1751688486456
---

There are three types of node components - they are the most important parts of a node:

    - [[sflo.concept.mesh.resource.element.node-component.metadata]] (mandatory)
    - [[sflo.concept.mesh.resource.element.node-component.reference]] (optional)
    - [[sflo.concept.mesh.resource.element.node-component.data]] (only for [[sflo.concept.mesh.resource.node.data]])

**Node components** are dcat:DatasetSeries representing abstract datasets about their node's metadata, referent, and payload data that exist through time, independent of any specific version or realization.

## Relationship to Node Components

As DatasetSeries, node components are realized through [[sflo.concept.mesh.resource.element.node-component.layer]], which are temporal concrete versions of the abstract concept. To borrow a phrase from the PROV model, we say that a layer is a specialization of the node component.

### Relationship pattern:

Every node component has at least two concrete layers: [[sflo.concept.mesh.resource.element.node-component.layer.current]] and [[sflo.concept.mesh.resource.element.node-component.layer.next]].

The node component is a [[related-topics.dcat.dataset-series]] and may have multiple component layers, i.e., [[sflo.concept.mesh.resource.element.node-component.layer.version]]s.

- node component: "My ontology definitions" (persistent concept)
- component layers: v1.0, v1.1, current version, working draft of next version (specific realizations)

### Ontology Example

```file
/my-ontology/
├── _ref-component/                    ← node component (reference data about ontology)
│   ├── _current/           ← component layer
│   ├── _v1/                ← component layer  
│   └── _v2/                ← component layer
└── _data-component/                  ← node component (ontology definitions)
    ├── _current/           ← component layer
    └── _v1/                ← component layer
```

In this example:

_ref/ represents the node component of reference information about the ontology
_data/ represents the node component of the ontology's definitional content
Each _current/, _v1/, etc. contains component layer realizations

## Persistent Identity

node components provide conceptual continuity by:

- Maintaining meaning across versions and changes
- Preserving references from external sources
- Enabling evolution while keeping identity stable
- Supporting versioning without losing conceptual coherence
