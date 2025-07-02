---
id: xmdevh3s6gvp93m3nyc6683
title: mesh resource
desc: ''
updated: 1751385422739
created: 1750709094321
---

## Overview

A **mesh resource** is any addressable component within a [[semantic mesh|sflow.concept.mesh]]. Every mesh resource has a unique [[identifier|sflow.concept.identifier]] based on its path and locally unique name, making it dereferenceable via URL.

In RDF terms, a resource is any node in an RDF graph that can be represented with an IRI (the other kinds of RDF graph nodes are literals and blank nodes).

## Types of Mesh Resources

The structure of a semantic mesh is built on a fundamental distinction between **extensible** and **terminal** resources:

### Mesh Nodes
**[[Mesh nodes|sflow.concept.mesh.resource.node]]** are extensible namespace containers:
- Extend the namespace and can be further extended by containing other mesh resources
- Always physically represented as folders

**Types of mesh nodes:**
- **[[Namespace nodes|sflow.concept.mesh.resource.node.namespace]]**: Empty containers for organizing other mesh nodes
- **[[Reference nodes|sflow.concept.mesh.resource.node.reference]]**: Nodes that refer to external entities and contain reference datasets
- **[[data nodes|sflow.concept.mesh.resource.node.data]]**: Nodes containing data distributions with optional versioning (includes [[dataset series|sflow.concept.mesh.resource.node.data.series]] which are data nodes whose metadata identifies them as DCAT dataset series)

### Mesh Elements
**[[Mesh elements|sflow.concept.mesh.resource.element]]** are terminal mesh resources:
- Extend the namespace but cannot be extended beyond their own internal structure
- All files and folders within an element folder are considerd to be part of the element
- Can be physically represented as folders or files

**Folder-based elements:**
- **[[Catalog datasets|sflow.concept.mesh.resource.element.meta-dataset]]**: Administrative metadata (in `_catalog/` folders)
- **[[Reference datasets|sflow.concept.mesh.resource.element.reference-dataset]]**: Referent data (in `_ref/` folders)
- **[[Asset trees|sflow.concept.mesh.resource.element.asset-tree]]**: File collections (in `_assets/` folders)
- **[[Version datasets|sflow.concept.mesh.resource.element.version-dataset]]**: Versioned snapshots
- **[[Next datasets|sflow.concept.mesh.resource.element.next-dataset]]**: Draft workspaces

**File-based elements:**
- **[[Resource pages|sflow.concept.mesh.resource.element.file.resource-page]]**: Generated index.html files
- **[[__unversion files|sflow.concept.mesh.resource.element.file.__unversion]]**: Version control sentinels
- **README.md and CHANGELOG.md**: Documentation files
- **[[Distribution files|sflow.concept.mesh.resource-facet.distribution]]**: Data files in RDF formats

## Physical vs Logical Structure

**Physical Representation:**
- Mesh nodes and elements are represented as folders in the filesystem
- File resources are represented as individual files
- Folder names become namespace segments and URL path components

**Logical Function:**
- All mesh resources are addressable via their URL path
- URLs must return meaningful content when dereferenced
- Resources maintain semantic relationships through containment and cross-references

## Asset Tree Special Case

[[Asset trees|sflow.concept.mesh.resource.element.asset-tree]] represent a special category where:
- The asset tree itself (with its [[catalog dataset|sflow.concept.mesh.resource.element.meta-dataset]]) is part of the mesh structure
- The files and folders contained within asset trees are "attached to" but not "contained in" the mesh
- Asset tree contents are addressable but are not considered semantic flow resources

This distinction maintains clean separation between semantic mesh structure and arbitrary file attachments while preserving addressability.