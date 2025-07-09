---
id: xmdevh3s6gvp93m3nyc6683
title: mesh resource
desc: ''
updated: 1752025355023
created: 1750709094321
---

## Overview

A **mesh resource** is any addressable component within a [[semantic mesh|sflo.concept.mesh]]. Every mesh resource has a unique [[identifier|sflo.concept.identifier]] based on its path and locally unique name, making it dereferenceable via URL.

In RDF terms, a resource is any node in an RDF graph that can be represented with an IRI (the other kinds of RDF graph nodes are literals and blank nodes). So theoretically, files and folders in [[sflo.concept.mesh.resource.element.asset-tree]] could be considered RDF resources. But they are not considered **mesh** resources

## Types of Mesh Resources

The structure of a semantic mesh is built on a fundamental distinction between **extensible** and **terminal** resources:

- **[[Mesh nodes|sflo.concept.mesh.resource.node]]** are extensible namespace containers:
- **[[Mesh elements|sflo.concept.mesh.resource.element]]** are terminal mesh resources:
  - Can be physically represented as folders or files
    - Folder [[sflo.concept.identifier]] are part of the namespace but cannot be extended beyond their own internal structure
  - All files and folders within an element folder are considerd to be part of the parent node

**Folder-based elements:**


- **[[Metadata components|sflo.concept.mesh.resource.element.node-component.metadata]]**: Administrative metadata (in `_meta-component/` folders)
- **[[Reference components|sflo.concept.mesh.resource.element.node-component.reference]]**: Referent data (in `_ref-component/` folders)
- **[[Asset trees|sflo.concept.mesh.resource.element.asset-tree]]**: File collections (in `_assets/` folders)
- **[[Version datasets|sflo.concept.mesh.resource.element.node-component.layer.version]]**: Versioned snapshots
- **[[Next layers|sflo.concept.mesh.resource.element.node-component.layer.next]]**: Draft workspaces

**File-based elements:**
- **Documentation files**: 
  - [[Resource pages|sflo.concept.mesh.resource.element.documentation-resource.resource-page]] are index.html files that provide de-referencability for their containing [[sflo.concept.identifier]] [[sflo.concept.mesh.resource-facet.folder]]
  - **README.md and CHANGELOG.md**: unstructured documentation
- **[[Layer distribution files|sflo.concept.mesh.resource.element.node-component.layer.distribution]]**: Data files in RDF formats

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

[[Asset trees|sflo.concept.mesh.resource.element.asset-tree]] represent a special category where:
- The asset tree itself (with its [[sflo.concept.mesh.resource.element.node-component.metadata]]) is part of the mesh structure
- The files and folders contained within asset trees are "attached to" but not "contained in" the mesh
- Asset tree contents are addressable but are not considered semantic flow resources

This distinction maintains clean separation between semantic mesh structure and arbitrary file attachments while preserving addressability.