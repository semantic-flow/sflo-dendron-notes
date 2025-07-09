---
id: h6ssv16gdyf56gg235dxv85
title: semantic mesh
desc: ''
updated: 1752025355019
created: 1750624002110
---

## Overview

A **semantic mesh** is a dereferenceable, possibly-versioned, [[sflo.concept.immutability]] collection of semantic data and other resources where every HTTP URL returns meaningful content. It serves as the foundational structure for organizing and publishing semantic web resources through [[semantic sites|sflo.concept.semantic-site]].

Key characteristics:
- **Addressable**: Every resource has a unique URL-based identifier
- **Dereferenceable**: All URLs return meaningful content when accessed
- **Versioned**: Changes are managed through the [[Weave Process|sflo.concept.weave-process]] process, and [[sflo.concept.mesh.resource.element.node-component]] are versioned by default
- **Publish-ready**: Can be served directly via GitHub Pages or similar static hosting; or via a local web server like live-server

## Core Concepts

### Mesh Resources

There are two primary categories:

#### Mesh Nodes

[[Mesh nodes|sflo.concept.mesh.resource.node]] are the primary structural components, physically represented as [[mesh folders|sflo.concept.mesh.resource-facet.folder]]. They extend namespaces and serve as containers.

- **[[Namespace nodes|sflo.concept.mesh.resource.node.namespace]]**: Empty containers for organizing other mesh nodes
- **[[Reference nodes|sflo.concept.mesh.resource.node.reference]]**: Nodes that refer to external entities (people, concepts, relationships) and contain [[reference components|sflo.concept.mesh.resource.element.node-component.reference]]
- **[[Data nodes|sflo.concept.mesh.resource.node.data]]**: Nodes containing data distributions with optional versioning


#### Elements

[[Mesh elements|sflo.concept.mesh.resource.element]] help define, support, and systematize nodes:

## Folder-based

- **[[sflo.concept.mesh.resource.element.node-component]]** and their [[sflo.concept.mesh.resource.element.node-component.layer]]
  - **[[sflo.concept.mesh.resource.element.node-component.metadata]]**: System-related administrative and structural metadata for mesh nodes
  - **[[Version datasets|sflo.concept.mesh.resource.element.node-component.layer.version]]**: Versioned snapshots of datasets
- **[[Next layers|sflo.concept.mesh.resource.element.node-component.layer.next]]**: Draft workspaces for ongoing changes to versioned datasets
- **[[Node handles|sflo.concept.mesh.resource.element.node-handle]]**: Elements that provide referential indirection, allowing references to nodes as mesh resources rather than their referents
- **[[Asset trees|sflo.concept.mesh.resource.element.asset-tree]]**: Collections of arbitrary files and folders attached to the mesh

#### Files

Terminal [[mesh resources|sflo.concept.mesh.resource]] that cannot contain other resources:

- **[[Resource pages|sflo.concept.mesh.resource.element.documentation-resource.resource-page]]**: index.html files present in every mesh folder after weaving
- **[[Distribution files|sflo.concept.mesh.resource.element.node-component.layer.distribution]]**: Data files in various RDF formats
- **README.md and CHANGELOG.md**: Documentation files providing context


## Physical Structure

### Folder Mapping
- Mesh nodes correspond physically to [[mesh folders|sflo.concept.mesh.resource-facet.folder]]
- Folder names become namespace segments and URL path components
- The local [[sflo.concept.identifier]] for a node matches its containing folder name

### File Organization
- [[Datasets|sflo.concept.mesh.resource-facet.dataset]] are represented by folders containing at least one distribution file
- Distribution files must be named using the dataset's [[namespace segment|sflo.concept.namespace.segment]]
- Resource pages (index.html) should be present in every mesh folder after [[weaving|sflo.concept.weave-process]]

### Reserved Names
- All reserved folder names begin with an underscore (_)
- Examples: `_assets/`, `_meta-component/`, `_ref-component/`, `_current`, `_next`

## Logical Structure

### Namespace Extension
- Mesh folders always extend the namespace with a segment corresponding to the folder name
- This creates a hierarchical URL structure for addressing resources
- Each resource has a unique [[identifier|sflo.concept.identifier]] based on its path and local name

### Containment Rules
- **Mesh nodes** are always containers of elements (i.e., at least [[sflo.concept.mesh.resource.element.node-component.metadata]] and [[sflo.concept.mesh.resource.folder._node-handle]]) and potentially containers of other nodes 
  - **namespace nodes**: no additional containment requirements
  - **reference nodes**: must have [[sflo.concept.mesh.resource.element.node-component.reference]]
  - **data nodes**: must have [[sflo.concept.mesh.resource.element.node-component.data]] with at least one distributions; and optionally, [[sflo.concept.mesh.resource.node.reference]]
- **Assets tree elements**: Cannot contain nodes
- all elements can contain 

## Rules & Constraints

### System vs User Boundaries
- **System elements**: Generated and managed by the weave process, not intended for user modification
- **User elements**: Directly modifiable by users ([[sflo.concept.mesh.resource.element.node-component.layer.current]], README.md, CHANGELOG.md)
- The weave process maintains system elements and generates missing required components

### Versioning Requirements
- Component versioning is managed through the [[Versioning|sflo.concept.versioning]] system
  - turning versioning on and off is controlled in the [[sflo.concept.mesh.resource.element.weave-config]]
  - Version history is realized in [[sflo.concept.mesh.resource.element.node-component.layer.version]] with numbered version snapshots
  - Version history metadata is kept in the node's [[sflo.concept.mesh.resource.element.node-component.metadata]]

### Addressing Requirements
- Every mesh resource must be addressable via its URL path
- URLs must return meaningful content when dereferenced
  - [[sflo.concept.mesh.resource.element.documentation-resource.resource-page]] provide human-readable information for [[sflo.concept.mesh.resource-facet.folder]]-based resources
    - resource pages are always index.html files generated by "on weave" from the [[sflo.concept.mesh.resource.element.documentation-resource.changelog]] and [[sflo.concept.mesh.resource.element.documentation-resource.readme]] [[sflo.concept.mesh.resource.element.documentation-resource]], templates in [[sflo.concept.mesh.resource.element.asset-tree]] and any scoped template mappings specified in [[sflo.concept.mesh.resource.element.weave-config]] files 
  - [[sflo.concept.mesh.resource-facet.file]]

## Integration Points

### Weave Process
The [[Weave Process|sflo.concept.weave-process]] process maintains mesh integrity by:
- Checking for required system resources and creating them if missing
- Generating resource pages for changed components
- Managing dataset versioning and metadata
- Ensuring all resources remain addressable and dereferenceable

### Publishing Workflow
- Meshes are designed to be served directly as static sites
- GitHub Pages integration allows immediate publishing after repository updates
- No static site generator required, though resource page generation occurs during weaving
- The repository structure directly maps to the published URL structure

### Dataset Integration
Meshes support multiple RDF formats and follow [[DCAT v3|related-topics.dcat.vocabulary]] standards for dataset organization. [[Datasets|sflo.concept.mesh.resource-facet.dataset]] within meshes include both standalone datasets and those embedded as mesh elements.
