---
id: 8t3swuswoi81yuzo2bnecy9
title: component layer
desc: ''
updated: 1752025354996
created: 1751689346769
---

**Component layers** are mesh elements that are datasets and represent the evolutionary steps of [[sflo.concept.mesh.resource.element.node-component]], whether [[sflo.concept.mesh.resource.element.node-component.metadata]], [[sflo.concept.mesh.resource.element.node-component.reference]], or [[sflo.concept.mesh.resource.element.node-component.data]]. 

Component layers have corresponding [[distributions|sflo.concept.mesh.resource.element.node-component.layer.distribution]] and are the connective tissue between nodes and their RDF-based representation.

## Relationship to Node Components

Component layers are the successive realizations of [[sflo.concept.mesh.resource.element.node-component]].

### Relationship pattern:

Node components have at least two layers:

- current version (`_current/`)
- working version (`_next`)

Versioned components 

### Ontology Data Node Example

```file
/my-ontology/               ← Data Node: Conceptual, data-oriented "thing"
├── _meta-component/                   ← Reference node component (reference data about ontology)
│   ├── _current/           ← component layer (current reference data)
│   ├── _next/              ← component layer (working draft)
│   ├── _v1/                ← component layer (version 1 reference data)
│   └── _v2/                ← component layer (version 2 reference data)
├── _ref-component/                   ← Reference node component (reference data about ontology)
│   ├── _current/           ← component layer (current reference data)
│   ├── _next/              ← component layer (working draft)
│   ├── _v1/                ← component layer (version 1 reference data)
│   └── _v2/                ← component layer (version 2 reference data)
└── _data-component/                  ← Data node component (ontology definition--by-dataset)
    ├── _current/           ← component layer (current definition)
    ├── _next/              ← component layer (working draft)
    └── _v1/                ← component layer (version 1 definition)
```

In this example:
- `_current/`, `_v1/`, `_v2/`, `_next/` are all component layers
- Each contains actual data files and distributions
- They represent specific temporal states of their parent node components

## Temporal Nature

component layers capture datasets at specific moments:

- **Current versions** (`_current/`) - The active working state
- **Next versions** (`_next/`) - Draft content for future release
- **Historical versions** (`_v1/`, `_v2/`) - Immutable snapshots from the past

## Content Structure

component layers contain:
- **Data files** - The actual dataset content (`.ttl`, `.rdf`, `.jsonld`)
- **Distributions** - Multiple format representations of the same data
- **Metadata** - Information about the specific version/snapshot

### Example Structure
```file
_current/
├── my-ontology.ttl         ← Distribution
├── my-ontology.rdf         ← Distribution  
└── my-ontology.jsonld      ← Distribution
```

## Immutability

**[[sflo.concept.mesh.resource.element.node-component.layer.version]]** (historical component layers, i.e., versioned folders like `_v1/`, `_v2/`) should be treated as immutable once created. This provides reliable references for external systems and ensures accurate provenance and history.

**[[sflo.concept.mesh.resource.element.node-component.layer.current]]** (the latest "woven" component layers, `_current`) should not be modified directly by users, but will be updated "on weave" if the [[sflo.concept.mesh.resource.element.node-component.layer.next]] has evolved. 

**[[sflo.concept.mesh.resource.element.node-component.layer.next]]** (working component layers, `_next/`) are mutable:
- Can be edited and updated during development
- Represent evolving state of the node component

## Creation and Lifecycle

component layers are created through:
- **Initial authoring** - Creating `_current/` content
- **Versioning** - Snapshotting `_current/` to `_v1/`, `_v2/` during [[sflo.concept.weave]]
- **Draft preparation** - Working in `_next/` for future releases

## Related Concepts

- **[[sflo.concept.mesh.resource.element.node-component]]** - Parent conceptual entities
- **[[sflo.concept.versioning]]** - Process of creating versioned component layers
- **[[sflo.concept.weave-process]]** - Operation that manages component layer lifecycle