---
id: p3mbdrze0qe8uvko4i16t1s
title: mesh folder
desc: ''
updated: 1750823850473
created: 1750659145476
---

- a mesh is structured with mesh folders, which correspond to RDF resources.
  When a mesh gets published, the folders also correspond to URL paths.
  - i.e., Folder = storage, Resource = identity

## Types

### User Mesh Folders

### System Mesh Folders

- **`_mesh/`**

  - Marks its parent folder as a **branch** mesh-node (namespace,
    reference-node, or SF-dataset).
  - **Branch-only**—only folders with `_mesh/` get recursed into when
    discovering nested user-nodes.
  - **Mutually exclusive** with all other system folders in the same directory.

- **`_assets/`**

  - Holds static user assets (images, CSS, binaries).
  - **Always terminal**—never recursed into by either the user-node or dataset
    walks.
  - Fully ignored by the mesh scanner.

- **`_ref/`**

  - Contains the **referent-metadata** SF-dataset for a reference-node (e.g.
    triples that say “this node proxies X”).
  - **User-terminal**—no child user-nodes beneath it.
  - **Recursed into** during the _dataset_ walk to pick up its distributions and
    any `_v-series/`.

- **`_v-series/`**

  - The “history closet” for a versioned SF-dataset (or `_ref/` dataset), hiding
    all `v1/`, `v2/`, … folders.
  - **User-terminal**—you list its version folders but do **not** recurse
    further.
  - Ensures the parent folder stays uncluttered even with lots of versions.

- **`v1/`, `v2/`, …**

  - Version snapshot folders inside `_v-series/`, each holding exactly one
    frozen distribution file (named `<BranchName>_vN.ext`).
  - **Fully terminal**—neither user-nodes nor system-folders may live inside.

### 

- **`_assets/`** Arbitrary sub-folders and files, to be included verbatim when
  publishing to a [[sflow.concept.site]].
  - Typically, they will supplement or complement the semantic content in the
    mesh.
  - **terminal**: i.e., they can't contain other mesh folders

## Rejected options

- directories: longer to type and most people just say folders
- URLs/IRIs: a mesh is local-first, and we don't use the "file:// schema" so it
  doesn't make sense to call these things URLs
- folder names are used for identifiers, but identifiers are contextualized with
  their containing path. if you're not talking about the whole path, it's really
  just a folder.
