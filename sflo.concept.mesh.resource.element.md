---
id: 9c27yly4ed3ju7msf8luhge
title: sflow element
desc: ''
updated: 1751697397021
created: 1750706813437
---

## Overview

**Mesh elements** are terminal mesh resources that support and define the mesh structure. Unlike [[mesh nodes|sflo.concept.mesh.resource.node]] which can contain other mesh resources, elements cannot be extended beyond their own internal structure.

Elements can be physically represented as folders or files, and all files and folders within an element folder are considered to be part of that element.

## Element Categories

Elements are categorized by their facets, including:
  - typical creation and maintenance patterns (user vs system)
  - verioning status
  - folder vs. file
  - node role (name, reference, and data [[sflo.concept.mesh.resource.element.abstract-dataset]])

### User Elements

User elements are primarily created and maintained by users or their software agents and services, and represent domain knowledge:

**Folder-based user elements:**
- **[[Asset trees|sflo.concept.mesh.resource.element.asset-tree]]**: Collections of arbitrary files attached to the mesh (in `_assets/` folders)
- **[[Next datasets|sflo.concept.mesh.resource.element.next-dataset]]**: Draft workspaces for ongoing changes to [[sflo.concept.mesh.resource.element.abstract-dataset]] (in `_next/` folders)

**File-based user elements:**
- **README.md files**: User documentation providing context
- **CHANGELOG.md files**: Version history documentation

### System Elements

System elements are usually created or altered by the [[Weave Process|sflo.concept.weave-process]] process rather than direct user modification:

**Folder-based system elements:**
- **[[Meta datasets|sflo.concept.mesh.resource.element.meta-dataset]]**: Administrative and structural metadata for mesh nodes (in `_meta/` folders)
- **[[Version datasets|sflo.concept.mesh.resource.element.version-dataset]]**: Versioned snapshots of datasets (in `_vN/` folders)
- **[[Node handles|sflo.concept.mesh.resource.element.node-handle]]**: Elements providing referential indirection for nodes as mesh resources (in `_handle/` folders)
- [[sflo.concept.mesh.resource.element.unified-dataset]]

**File-based system elements:**
- **[[Resource pages|sflo.concept.mesh.resource.element.resource-page]]**: Generated index.html files for human-readable access
- **[[Distribution files|sflo.concept.mesh.resource-facet.distribution]]**: Data files in various RDF formats

## Physical vs Logical Structure

**Physical Representation:**
- Folder-based elements are represented as folders with underscore prefixes (like `_meta/`, `_ref/`, `_assets/`)
- File-based elements are individual files within mesh nodes or other elements
- Element folders contain all files and folders that belong to that element

**Logical Function:**
- Elements extend the namespace but are terminal (cannot contain other mesh nodes or elements)
- Elements provide specialized functionality: metadata, versioning, referential data, or file attachments
- Elements maintain the semantic structure and operational capabilities of the mesh

## Integration with Nodes

Elements work in conjunction with mesh nodes to create the complete mesh structure:
- Every mesh node contains system elements (meta datasets, node handles)
- Reference nodes contain reference datasets (user elements)
- data nodes may contain versioning elements (system elements)
- Any node may contain asset trees (user elements) for file attachments
