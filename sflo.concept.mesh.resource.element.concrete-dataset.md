---
id: 8t3swuswoi81yuzo2bnecy9
title: concrete dataset
desc: ''
updated: 1751689687965
created: 1751689346769
---

Concrete datasets are mesh resource elements that represent specific instances of [[abstract datasets|sflo.concept.mesh.resource.element.abstract-dataset]]. They have actual data distributions and correspond to different points in an dataset's evolution.

## Relationship to Abstract Datasets

Concrete datasets are realizations of [[abstract datasets|sflo.concept.mesh.resource.element.abstract-dataset]], which represent the persistent conceptual identity of a dataset through time.

### Relationship pattern:

One abstract dataset may have multiple concrete datasets, e.g.:

- Abstract dataset: "My ontology definitions" (`_data/`)
- Concrete datasets: current working version (`_current/`), version 1.0 (`_v1/`), version 2.0 (`_v2/`)

### Ontology Example

```file
/my-ontology/
├── _ref/                    ← Abstract dataset (reference data about ontology)
│   ├── _current/           ← Concrete dataset (current reference data)
│   ├── _v1/                ← Concrete dataset (version 1 reference data)
│   └── _v2/                ← Concrete dataset (version 2 reference data)
└── _data/                  ← Abstract dataset (ontology definitions)
    ├── _current/           ← Concrete dataset (current definitions)
    ├── _next/              ← Concrete dataset (working draft)
    └── _v1/                ← Concrete dataset (version 1 definitions)
```

In this example:
- `_current/`, `_v1/`, `_v2/`, `_next/` are all concrete datasets
- Each contains actual data files and distributions
- They represent specific temporal states of their parent abstract datasets

## Temporal Nature

Concrete datasets capture datasets at specific moments:

- **Current versions** (`_current/`) - The active working state
- **Next versions** (`_next/`) - Draft content for future release
- **Historical versions** (`_v1/`, `_v2/`) - Immutable snapshots from the past

## Content Structure

Concrete datasets contain:
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

**Historical concrete datasets** (versioned folders like `_v1/`, `_v2/`) are immutable once created:
- Content cannot be modified after versioning
- Provides reliable references for external systems
- Maintains dataset provenance and history

**Working concrete datasets** (`_current/`, `_next/`) are mutable:
- Can be edited and updated during development
- Represent evolving state of the abstract dataset

## WEMI Alignment

In the Work/Expression/Manifestation/Item hierarchy:
- **Expression** → Concrete dataset (`_current/`, `_v1/`)
- **Manifestation** → Distribution files (`.ttl`, `.rdf`)
- **Item** → Actual files on storage medium

Concrete datasets represent specific expressions of the abstract work, realized through various manifestations (file formats).

## Creation and Lifecycle

Concrete datasets are created through:
- **Initial authoring** - Creating `_current/` content
- **Versioning** - Snapshotting `_current/` to `_v1/`, `_v2/` during [[sflo.concept.weave]]
- **Draft preparation** - Working in `_next/` for future releases

## Related Concepts

- **[[sflo.concept.mesh.resource.element.abstract-dataset]]** - Parent conceptual entities
- **[[sflo.concept.dataset-versioning]]** - Process of creating versioned concrete datasets
- **[[sflo.concept.weave-process]]** - Operation that manages concrete dataset lifecycle