---
id: rregmt56znauz71qgypet6a
title: abstract dataset
desc: ''
updated: 1751690361352
created: 1751688486456
---

Abstract datasets represents the "dataset as concept", and exist through time, independent of any specific version, snapshot, or temporal realization.

They are 

## Relationship to Dataset Series

## Relationship to Concrete Datasets

Abstract datasets are realized through [[concrete datasets|sflo.concept.mesh.resource.element.concrete-dataset]], which are specific temporal snapshots or versions of the abstract concept.

### Relationship pattern:

One abstract dataset may have multiple concrete datasets, e.g.:

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

## WEMI Alignment

Abstract datasets represent the intellectual or conceptual "work" that can be expressed in multiple concrete forms over time. Distributions of a concrete dataset correspond to manifestations, and the actual files on a particular storage medium would correspond to item.