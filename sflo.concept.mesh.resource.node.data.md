---
id: 8hmkiyjtsey7z8y5oi5xdxx
title: data node
desc: ''
updated: 1752697259224
created: 1750999795528
---

## Overview

**Data nodes** are [[mesh nodes|sflo.concept.mesh.resource.node]] that represent a data concept, have an abstract "payload" dataset in the form of an [[sflo.concept.mesh.resource.element.flow.data]] (an [[sflo.concept.mesh.resource.element.flow]]). The node data flow is in turn instantiated by conceptual [[sflo.concept.mesh.resource.element.flow.snapshot]]s that represent the data's evolution and current state.

Unlike [[dataset elements|sflo.concept.mesh.resource.element.flow.snapshot]] which contain concrete data distributions, data nodes serve as conceptual containers that organize and provide identity for data without containing the data directly. I.e., Data nodes only contain concrete datasets by virtue of containing [[sflo.concept.mesh.resource.element.flow.data]] (also abstract) and its layers, which have concrete distributions.

Data nodes are physically represented as [[mesh folders|sflo.concept.mesh.resource-facet.folder]] and correspond to [[namespace segments|sflo.concept.namespace.segment]].

## Abstract vs Concrete Data

### Abstract Data Concept (Data Node)
A data node represents the **idea** or **concept** represented by a dataset:
- `/ns/djradon/bio/` = a biographical dataset about the person djradon
- `/ns/census/` =  the results of a census
- `/ns/weather-stations/` = "the concept of weather station data"
This idea or concept is the referent of the data node's URL. 

The data node provides:
- **Stable identity**: The concept persists even as concrete data changes
- **Organizational structure**

### data flows (Dataset Elements)

[[sflo.concept.mesh.resource.element.flow.data]] refer to themselves and contain layers:

- `/ns/monsters/_data/_v4/` = "the current monster data flow"
- `/ns/weather-stations/_v3/` = "version 3 of weather station data"

These elements contain:
- **Distribution files**: The actual data in various formats
- **Concrete metadata**: Information about this specific data instance

## Required Structure

Every data node must contain:

- **[[sflo.concept.mesh.resource.element.flow.metadata]]** (`_meta-component/`): Administrative metadata about the data concept
- **[[sflo.concept.mesh.resource.element.flow.data]]** (`_data-component/`): dataset data
- **[[Node handle|sflo.concept.mesh.resource.element.node-handle]]** (`_node-handle/`): Referential indirection for the node


## Optional Structure

- **[[Asset trees|sflo.concept.mesh.resource.element.asset-tree]]** (`_assets/`): Attached file collections
- [[sflo.concept.mesh.resource.element.documentation-resource.changelog]] and [[sflo.concept.mesh.resource.element.documentation-resource.readme]]
- [[sflo.concept.mesh.resource.element.node-config-defaults]]
- [[sflo.concept.mesh.resource.element.flow.unified]] 

## Key Characteristics

### Not a Dataset

**Important**: A data node does not refer to a specifc RDF graph; it is **not itself a (concrete) dataset**. It represents the abstract concept of a dataset that may evolve over time:
- Data nodes are never versioned (only their elements are)
- Data nodes serve as stable conceptual anchors

### Overlap with Reference Nodes

Data nodes function like [[reference nodes|sflo.concept.mesh.resource.node.reference]], in that they refer to something... but their referent is very specific: an (abstract) dataset that may evolve over time.

### Extensible Container
Like all mesh nodes, data nodes can contain other mesh nodes and elements, making them extensible namespace containers.

## Examples

### Unversioned Data Node
```
ns/monsters/
├── _meta-component/        # metadata about an abstract "monsters dataset"
├── _node-handle/           # handle for the data node
└── _current/               # current monster data
    ├── monsters.jsonld     # concrete distribution of the current monster snapshot
    └── monsters.ttl
```

