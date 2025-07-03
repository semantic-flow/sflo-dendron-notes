---
id: 8hmkiyjtsey7z8y5oi5xdxx
title: data node
desc: ''
updated: 1751565550573
created: 1750999795528
---

## Overview

**Data nodes** are [[mesh nodes|sflo.concept.mesh.resource.node]] that represent abstract data concepts. Unlike [[dataset elements|sflo.concept.mesh.resource.element]] which contain concrete data distributions, data nodes serve as conceptual containers that organize and provide identity for data without containing the data directly.

Data nodes are physically represented as [[mesh folders|sflo.concept.mesh.resource.folder]] and correspond to [[namespace segments|sflo.concept.namespace.segment]].

## Abstract vs Concrete Data

### Abstract Data Concept (Data Node)
A data node represents the **idea** or **concept** of a dataset:
- `/ns/monsters/` = "the concept of monster data"
- `/ns/census/` = "the concept of census data"
- `/ns/weather-stations/` = "the concept of weather station data"

The data node provides:
- **Stable identity**: The concept persists even as concrete data changes
- **Metadata container**: Information about the data concept itself
- **Organizational structure**: A place to organize temporal dataset elements

### Concrete Data Instance (Dataset Elements)
[[Temporal dataset elements|sflo.concept.mesh.resource.element.temporal-dataset]] contain the actual data:
- `/ns/monsters/_current/` = "the current monster data files"
- `/ns/census/_current/` = "the current census data files"  
- `/ns/weather-stations/_v3/` = "version 3 of weather station data"

These elements contain:
- **Distribution files**: The actual data in various formats
- **Concrete metadata**: Information about this specific data instance

## Required Structure

Every data node must contain:

### System Elements (Required)
- **[[meta dataset|sflo.concept.mesh.resource.element.meta-dataset]]** (`_catalog/`): Administrative metadata about the data concept
- **[[Node handle|sflo.concept.mesh.resource.element.node-handle]]** (`_handle/`): Referential indirection for the node
- **[[Current dataset|sflo.concept.mesh.resource.element.current-dataset]]** (`_current/`): The current concrete data instance

### Optional Elements
- **[[Next dataset|sflo.concept.mesh.resource.element.next-dataset]]** (`_next/`): Draft workspace for changes
- **[[V-series dataset series|sflo.concept.mesh.resource.element.v-series-dataset-series]]** (`_v-series/`): Version history management
- **[[Version datasets|sflo.concept.mesh.resource.element.version-dataset]]** (`_v1/`, `_v2/`, etc.): Historical data instances
- **[[Asset trees|sflo.concept.mesh.resource.element.asset-tree]]** (`_assets/`): Attached file collections

### Special Files
- **`__unversion`**: Sentinel file that prevents version creation (lives at data node level)

## Key Characteristics

### Not a Dataset
**Important**: A data node is **not itself a dataset**. It represents the abstract concept of data but contains no distributions directly. This means:
- Data nodes are never versioned (only their elements are)
- Data nodes don't have distribution files at their root level
- Data nodes serve as stable conceptual anchors

### Parallel to Reference Nodes
Data nodes follow the same pattern as [[reference nodes|sflo.concept.mesh.resource.node.reference]]:
- **Reference nodes**: Contain required `_ref/` element for external entity data
- **Data nodes**: Contain required `_current/` element for current data instance

### Extensible Container
Like all mesh nodes, data nodes can contain other mesh nodes and elements, making them extensible namespace containers.

## Examples

### Simple Data Node
```
ns/monsters/
├── _catalog/           # metadata about "monsters as a data concept"
├── _handle/           # handle for the data node
├── _current/          # current monster data
│   ├── monsters.jsonld
│   └── monsters.ttl
└── __unversion        # prevent versioning
```

### Versioned Data Node
```
ns/census/
├── _catalog/          # metadata about "census as a data concept"
├── _handle/          # handle for the data node  
├── _current/         # current census data
├── _next/            # draft changes
├── _v-series/        # version history metadata
├── _v1/              # 2020 census data
└── _v2/              # 2021 census data
```

## Integration with Mesh

Data nodes work in conjunction with other mesh components:
- Provide stable identity for data concepts
- Organize temporal dataset elements
- Enable clean separation between abstract concepts and concrete data
- Support the [[single referent principle|sflo.principle.single-referent]] by having clear semantic boundaries
