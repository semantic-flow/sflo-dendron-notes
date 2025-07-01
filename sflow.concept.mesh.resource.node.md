---
id: 8hmkiyjtsey7z8y5oi5xdxm
title: mesh node
desc: ''
updated: 1751385422741
created: 1750999795528
---

## Overview

The primary constituents of a semantic mesh are **mesh nodes**. They are physically represented as [[mesh folders|sflow.concept.mesh.resource.folder]] and they correspond to [[namespace segments|sflow.concept.namespace.segment]].

Mesh nodes are extensible namespace containers that can contain other mesh nodes and [[mesh elements|sflow.concept.mesh.resource.element]], distinguishing them from elements which are terminal within their own scope.

## Types

There are three types of mesh nodes:

### Namespace Nodes
**[[Namespace nodes|sflow.concept.mesh.resource.node.namespace]]** are containers for other mesh nodes:
- Function primarily as organizational containers
- Do not contain primary data themselves, although they have catalogs for their metadata
- Can contain any other type of mesh node

### Reference Nodes  
**[[Reference nodes|sflow.concept.mesh.resource.node.reference]]** refer to external entities:
- Represent things other than themselves (people, concepts, relationships)
- Physically represented with folders that contain a [[reference dataset|sflow.concept.mesh.resource.element.reference-dataset]]
- Can contain other mesh nodes in addition to their reference dataset

### data nodes
**[[data nodes|sflow.concept.mesh.resource.node.data]]** contain data distributions:
- Have distributions and optional versioning capabilities
- Include a [[catalog dataset|sflow.concept.mesh.resource.element.catalog-dataset]] for metadata
- May contain other mesh nodes
- Can be configured as [[dataset series|sflow.concept.mesh.resource.node.data.series]] through their metadata

## Physical Structure

All mesh nodes:
- Are physically represented as folders in the filesystem
- Have folder names that become namespace segments
- Extend the URL namespace with their folder name
- Can be further extended by containing other mesh resources

## Common Elements

Every mesh node contains:
- A [[node handle|sflow.concept.mesh.resource.element.node-handle]] for referential indirection
- A [[catalog dataset|sflow.concept.mesh.resource.element.catalog-dataset]] for administrative metadata
- Optional additional elements depending on the node type