---
id: 9c27yly4ed3ju7msf8luhge
title: sflow element
desc: ''
updated: 1751385422742
created: 1750706813437
---

## Overview

**Mesh elements** are terminal mesh resources that support and define the mesh structure. Unlike [[mesh nodes|sflow.concept.mesh.resource.node]] which can contain other mesh resources, elements cannot be extended beyond their own internal structure.

Elements can be physically represented as folders or files, and all files and folders within an element folder are considered to be part of that element.

## Element Categories

Elements are categorized by their typical creation and maintenance patterns:

### User Elements
User elements are primarily created and maintained by users, representing domain knowledge:

**Folder-based user elements:**
- **[[Reference datasets|sflow.concept.mesh.resource.element.reference-dataset]]**: Data about a reference node's referent (in `_ref/` folders)
- **[[Asset trees|sflow.concept.mesh.resource.element.asset-tree]]**: Collections of arbitrary files attached to the mesh (in `_assets/` folders)

**File-based user elements:**
- **README.md files**: User documentation providing context
- **CHANGELOG.md files**: Version history documentation

### System Elements
System elements are usually created or altered by the [[weave|sflow.concept.weave]] process rather than direct user modification:

**Folder-based system elements:**
- **[[Catalog datasets|sflow.concept.mesh.resource.element.catalog-dataset]]**: Administrative and structural metadata for mesh nodes (in `_catalog/` folders)
- **[[Version datasets|sflow.concept.mesh.resource.element.version-dataset]]**: Versioned snapshots of datasets (in `_vN/` folders)
- **[[V-series dataset series|sflow.concept.mesh.resource.element.v-series-dataset-series]]**: Collections of version datasets that track historical checkpoints (in `_v-series/` folders)
- **[[Next datasets|sflow.concept.mesh.resource.element.next-dataset]]**: Draft workspaces for ongoing changes to versioned datasets (in `_next/` folders)
- **[[Node handles|sflow.concept.mesh.resource.element.node-handle]]**: Elements providing referential indirection for nodes as mesh resources (in `_handle/` folders)

**File-based system elements:**
- **[[Resource pages|sflow.concept.mesh.resource.element.file.resource-page]]**: Generated index.html files for human-readable access
- **[[__unversion files|sflow.concept.mesh.resource.element.file.__unversion]]**: Sentinel files that prevent version creation during weaving
- **[[Distribution files|sflow.concept.mesh.resource-facet.distribution]]**: Data files in various RDF formats

## Physical vs Logical Structure

**Physical Representation:**
- Folder-based elements are represented as folders with underscore prefixes (like `_catalog/`, `_ref/`, `_assets/`)
- File-based elements are individual files within mesh nodes or other elements
- Element folders contain all files and folders that belong to that element

**Logical Function:**
- Elements extend the namespace but are terminal (cannot contain other mesh nodes or elements)
- Elements provide specialized functionality: metadata, versioning, referential data, or file attachments
- Elements maintain the semantic structure and operational capabilities of the mesh

## Integration with Nodes

Elements work in conjunction with mesh nodes to create the complete mesh structure:
- Every mesh node contains system elements (catalog datasets, node handles)
- Reference nodes contain reference datasets (user elements)
- data nodes may contain versioning elements (system elements)
- Any node may contain asset trees (user elements) for file attachments
