---
id: 9zlt3thtzusgsq1ype2e2ko
title: metadata facet
desc: ''
updated: 1751438050755
created: 1751389633373
---

# Metadata Facet

## Overview

The metadata facet encompasses different types of metadata maintained across the four-facet node architecture. Each facet manages its own specific kind of metadata with clear boundaries between system and domain concerns.

## Four-Facet Metadata Architecture

```
node/
├── _name/            # Name metadata (mandatory, domain concerns)
├── _meta/            # Mesh/node metadata (mandatory, system concerns)  
├── _ref/             # Referent metadata (optional, domain concerns)
└── _data/            # Data metadata (optional, domain concerns)
```

## Metadata Types

### Name Metadata (`_name/`)
- **Mandatory**: Every node must have a name - names are fundamental for bringing things into existence
- **Philosophical Importance**: Names give identity and enable reference, making the abstract concrete
- **Domain Focus**: What this name means and how it's used across contexts
- **Contents**: Backlinks/mentions, sameAs assertions, name authority and provenance, name evolution, identity relationships

### Mesh Metadata (`_meta/`)  
- **Mandatory**: Every node needs system management within the mesh structure
- **System Focus**: How this node operates and relates within the mesh infrastructure
- **Contents**: Handles, mesh structure relationships, versioning coordination, system-level connections

### Referent Metadata (`_ref/`)
- **Optional**: Only for nodes that reference external entities
- **Domain Focus**: Provenance and reliability of reference information
- **Contents**: Reference source validation, authority chains, entity identification quality

### Data Metadata (`_data/`)
- **Optional**: Only for nodes that contain datasets
- **Domain Focus**: Provenance and characteristics of dataset content  
- **Contents**: Data quality metrics, processing history, schema definitions, lineage information

## Architectural Benefits

**Ontological Foundation**: Names serve as the fundamental building blocks that bring nodes into existence

**Clear System vs Domain Separation**: `_meta/` handles infrastructure while other facets focus on meaning

**Beautiful Symmetry**: Two mandatory facets (identity + system) and two optional facets (reference + data)

**Conceptual Clarity**: Each facet has distinct concerns that don't overlap
