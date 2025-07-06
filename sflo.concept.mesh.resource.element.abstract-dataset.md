---
id: rregmt56znauz71qgypet6a
title: abstract dataset
desc: ''
updated: 1751774485091
created: 1751688486456
---

Abstract datasets represents the "dataset as concepts", and exist through time, independent of any specific version, snapshot, or temporal realization.

## Relationship to Concrete Datasets

Abstract datasets are realized through [[concrete datasets|sflo.concept.mesh.resource.element.concrete-dataset]], which are specific temporal snapshots or versions of the abstract concept. To borrow a phrase from the PROV model, we say that concrete dataset is a specialization of the abstract dataset."

### Relationship pattern:

Every abstract dataset has at least two concrete datasets: [[sflo.concept.mesh.resource.element.current-dataset]] and [[sflo.concept.mesh.resource.element.next-dataset]].

An abstract dataset, if versioned, is a [[related-topics.dcat.dataset-series]] and may have multiple concrete datasets, i.e., [[sflo.concept.mesh.resource.element.version-dataset]]s.

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
