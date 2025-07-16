---
id: 8hmkiyjtsey7z8y5oi5xdxm
title: MeshNode
desc: ''
updated: 1752025355003
created: 1750999795528
---

## Overview

The primary constituents of a semantic mesh are **mesh nodes**. They are physically represented as [[mesh folders|sflo.concept.mesh.resource-facet.folder]] and provide one type of [[namespace segments|sflo.concept.namespace.segment]].

Mesh nodes are extensible namespace containers that can contain other mesh nodes and [[mesh elements|sflo.concept.mesh.resource.element]], distinguishing them from elements which are terminal within their own scope.

## Physical Structure

When stored on disk, all mesh nodes:
- Are physically represented as folders in the filesystem
- Have folder names that become namespace segments
- Extend the URL namespace with their folder name
- Can be further extended by containing other mesh resources

## Mandatory Elements

Every mesh node has these elements:

- **[[sflo.concept.mesh.resource.element.flow.metadata]]** (`_meta-component/`): Centralized metadata for the node
- **[[sflo.concept.mesh.resource.element.handle]]** (`_node-handle/`): Universal marker folder that refers to the parent "as a mesh node", as opposed to "as the name, dataset, or other thing" to which it normally refers; a handle resource page should explain this distinction

## Node Types

### 1. [[Namespace Node|sflo.concept.mesh.resource.node.namespace]]
**Elements**: `_meta-component/` + `_node-handle/`
- Functions as organizational containers
- Contains essential identity, metadata, and handle information
- Node IRI refers to the namespace itself
- Base level for all mesh nodes

### 2. [[Reference Node|sflo.concept.mesh.resource.node.reference]]
**Elements**: `_meta-component/` + `_node-handle/` + `_ref-component/`
- Represents external entities (people, concepts, relationships)
- Node IRI refers to the external entity being referenced
- Adds reference data capabilities to the namespace foundation
- Evolved from namespace nodes by adding the `_ref-component/` element
- Maintains single referent principle - the node refers to the external entity

### 3. [[Data Node|sflo.concept.mesh.resource.node.data]]
**Elements**: `_meta-component/` + `_node-handle/` + `_data-component/` ( + optional `_ref-component`)
- Contains data distributions and versioning capabilities
- Node IRI refers to the node flow
- Adds dataset storage to the namespace foundation
- Can be configured as [[dataset series|sflo.concept.mesh.resource.node.data.series]]
- Evolved from namespace nodes by adding the `_data-component/` element
- Maintains single referent principle - the node refers to the dataset
