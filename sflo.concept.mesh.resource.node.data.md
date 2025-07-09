---
id: 8hmkiyjtsey7z8y5oi5xdxx
title: data node
desc: ''
updated: 1752025355027
created: 1750999795528
---

## Overview

**Data nodes** are [[mesh nodes|sflo.concept.mesh.resource.node]] that represent a data concept, have abstract "payload" datasets in the form of an [[sflo.concept.mesh.resource.element.node-component.data]] (an [[sflo.concept.mesh.resource.element.node-component]]). The node component is in turn instantiated by [[sflo.concept.mesh.resource.element.node-component.layer]]s that represent the data's evolution and current state.

Unlike [[dataset elements|sflo.concept.mesh.resource.element]] which contain concrete data distributions, data nodes serve as conceptual containers that organize and provide identity for data without containing the data directly.

Data nodes are physically represented as [[mesh folders|sflo.concept.mesh.resource-facet.folder]] and correspond to [[namespace segments|sflo.concept.namespace.segment]].

## Abstract vs Concrete Data

### Abstract Data Concept (Data Node)
A data node represents the **idea** or **concept** represented by a dataset:
- `/ns/djradon-bio/` = biographical information about the person djradon
- `/ns/census/` =  the results of a census
- `/ns/weather-stations/` = "the concept of weather station data"
This idea or concept is the referent of the data node's URL. 

The data node provides:
- **Stable identity**: The concept persists even as concrete data changes
- **Organizational structure**

### Concrete Data Instance (Dataset Elements)
[[Temporal dataset elements|sflo.concept.mesh.resource.element.temporal-dataset]] contain the actual data:
- `/ns/monsters/_data/_v4/` = "the current monster data component"
- `/ns/census/_ref/_current` = "the current reference component which describes the referent", i.e., the particular census
- `/ns/weather-stations/_v3/` = "version 3 of weather station data"

These elements contain:
- **Distribution files**: The actual data in various formats
- **Concrete metadata**: Information about this specific data instance

## Required Structure

Every data node must contain:

- **[[sflo.concept.mesh.resource.element.node-component.metadata]]** (`_meta-component/`): Administrative metadata about the data concept
- **[[sflo.concept.mesh.resource.element.node-component.data]]** (`_data-component/`): dataset data
- **[[Node handle|sflo.concept.mesh.resource.element.node-handle]]** (`_node-handle/`): Referential indirection for the node


## Optional Structure

- **[[sflo.concept.mesh.resource.element.node-component.reference]]** (`_ref-component/`): reference data
- **[[Asset trees|sflo.concept.mesh.resource.element.asset-tree]]** (`_assets/`): Attached file collections
- [[sflo.concept.mesh.resource.element.documentation-resource.changelog]] and [[sflo.concept.mesh.resource.element.documentation-resource.readme]]
- [[sflo.concept.mesh.resource.element.weave-config]]
- [[sflo.concept.mesh.resource.element.unified-dataset]] 

## Key Characteristics

### Not a Dataset

**Important**: A data node is **not itself a dataset**. It represents the abstract concept of data:
- Data nodes are never versioned (only their elements are)
- Data nodes serve as stable conceptual anchors



### Overlap with Reference Nodes

Data nodes function like [[reference nodes|sflo.concept.mesh.resource.node.reference]], just with assoicated data
- **Reference nodes**: Contain required `_ref-component/` element for external entity data

### Extensible Container
Like all mesh nodes, data nodes can contain other mesh nodes and elements, making them extensible namespace containers.

## Examples

### Unversioned Data Node
```
ns/monsters/
├── _meta-component/           # metadata about "monsters as a data concept"
├── _node-handle/           # handle for the data node
└── _current/          # current monster data
    ├── monsters.jsonld
    └── monsters.ttl
```

