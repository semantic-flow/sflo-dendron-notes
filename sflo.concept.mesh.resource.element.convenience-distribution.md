---
id: e6enrj5ztz3hz84ojujef0k
title: Convenience Distribution
desc: ''
updated: 1751632665736
created: 1751631229565
---

---
id: [generated-id]
title: Convenience Distribution
desc: 'Node-level convenience datasets for clean external URLs'
updated: [timestamp]
created: [timestamp]
---

A **convenience distribution**, for [[sflo.concept.mesh.resource.node.data]] and [[sflo.concept.mesh.resource.node.reference]] only, is a hard copy of the relevant dataset's ([[sflo.concept.mesh.resource.element.data-dataset]] and [[sflo.concept.mesh.resource.element.reference-dataset]]) respective current distributions, placed directly at the node level to provide clean, standard URLs that external tools and users expect, without exposing mesh-specific structure.

## Purpose

Convenience distributions solve the tension between:
- **Mesh design requirements** - Structured folders like `_data/_current/` for internal consistency
- **External expectations** - Clean URLs like `ontology.ttl` that follow semantic web conventions
- **Static hosting constraints** - No server-side redirects or content negotiation

## Implementation

Convenience distributions are **hard copies** created during [[sflo.concept.weave-process]] by copying content from canonical locations to node-level paths.

### For Data Nodes
```
/my-ontology/
├── my-ontology.trig           ← Convenience distribution (copied)
├── my-ontology.jsonld        ← Convenience distribution (copied)
├── index.html                ← Node resource page
├── _data/
│   └── _current/
│       ├── my-ontology.trig   ← Canonical source
│       └── my-ontology.jsonld ← Canonical source
└── _unified/                 ← Full mesh metadata version
    └── my-ontology_unified.ttl
```

### For Reference Nodes
```
/person-alice/
├── person-alice.trig          ← Convenience distribution (copied)
├── person-alice.jsonld          ← Convenience distribution (copied)
├── index.html                ← Node resource page
├── _ref/
│   └── _current/
│       └── person-alice.ttl  ← Canonical source
└── _unified/
    └── person-alice_unified.ttl
```

## URL Benefits

**External tools see clean URLs:**
- `https://example.com/my-ontology/my-ontology.ttl`
- `https://example.com/person-alice/person-alice.ttl`

**Instead of mesh-specific paths:**
- `https://example.com/my-ontology/_data/_current/my-ontology.ttl`
- `https://example.com/person-alice/_ref/_current/person-alice.ttl`

## Weaving Process

During [[sflo.concept.weave-process]], convenience distributions are:

1. **Identified** - Determine which datasets need convenience copies
2. **Copied** - Hard copy from canonical location to node level
3. **Validated** - Confirm file integrity and naming consistency

## Source of Truth

- **Canonical source** - Always the `_current/` version in appropriate [[sflo.concept.mesh.resource.folder.dataset]] folder
  - Read-only copy, never edited directly
- **Regeneration** - Convenience copies recreated "on weave" if the relevant dataset has changed

## Use Cases

### Ontology Publishing
Enables standard ontology version IRIs that return actual ontology content:
```
https://semantic-flow.github.io/sflo-ontology/sflo-ontology.ttl
```

### API Compatibility
Provides URLs that work with existing RDF tools and libraries expecting standard file extensions.

### Human Usability
Offers intuitive URLs that match user expectations for downloadable semantic data.

## Relationship to Other Concepts

- **[[sflo.concept.mesh.resource.element.data-dataset]]** - Source for data node convenience distributions
- **[[sflo.concept.mesh.resource.element.reference-dataset]]** - Source for reference node convenience distributions  
- **[[sflo.concept.mesh.resource.element.unified-dataset]]** - Contains full mesh metadata, distinct from convenience distributions
- **[[sflo.concept.weave-process]]** - Process that creates and maintains convenience distributions

## Constraints

- **No symlinks** - GitHub Pages doesn't support symlinks, requiring hard copies
- **Name matching** - Convenience distribution filename must match containing folder name
- **Metadata** - both copies should be described in the node's metadata

## Storage Trade-offs

- **Duplication** - Each format exists in both canonical and convenience locations
- **Sync overhead** - Weaving process must maintain multiple copies