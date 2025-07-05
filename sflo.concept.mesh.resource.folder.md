---
id: p3mbdrze0qe8uvko4i16t1s
title: mesh folder
desc: ''
updated: 1751747159603
created: 1750659145476
---

- a mesh is structured with mesh folders, which correspond to RDF resources.
  When a mesh gets published, the folders also correspond to URL paths.
  - i.e., Folder = storage, Resource = identity

## Types

### User Mesh Folders

- **`_current`** 

### System Mesh Folders

- **`_meta/`**
  - present in mesh nodes and [[sflo.concept.mesh.resource.element.asset-tree]]

- **`_ref/`**

  - Contains the **referent data** for [[reference-node|sflo.concept.mesh.resource.node.reference]] and [[sflo.concept.mesh.resource.node.data]] (i.e., triples that say things about the thing the node represents).

- **`v1/`, `v2/`, …**

  - Version snapshot folders that represent [[sflo.concept.mesh.resource.element.concrete-dataset]]
  - each holds one or more distribution file (named `<node_ref_vN.ext`).
  - **Fully terminal**—neither user-nodes nor system-folders may live inside.

### User Mesh Folders

- **`_assets/`**

  - Holds static user assets (images, CSS, binaries).
  - **Always terminal** - never contains nodes
  - Except for its `_meta` folder, is ignored by the mesh scanner.

## Rejected options

- directories: longer to type and most people just say folders
- URLs/IRIs: a mesh is local-first, and we don't use the "file:// schema" so it
  doesn't make sense to call these things URLs
- folder names are used for identifiers, but identifiers are contextualized with
  their containing path. if you're not talking about the whole path, it's really
  just a folder.
