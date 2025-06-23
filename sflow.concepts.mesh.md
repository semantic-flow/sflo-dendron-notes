---
id: h6ssv16gdyf56gg235dxv85
title: semantic mesh
desc: ''
updated: 1750715874953
created: 1750624002110
---

A **semantic mesh** is a tree-shaped structure for organizing [[t.cs.semantic-web]] resources. It is composed of three main elements: identifier folders, namespace folders, and dataset folders. 

Meshes make it easy to discover, validate, and process your [[Linked Data|t.cs.web.linked-data]] without a heavyweight database or index.

It's simplest embodiment is as a "directory tree" on disk: just a bunch of nested folders with files.

### Purpose

A semantic mesh can be highly useful by itself, even as just some local files. But when published, shared, and re-woven, it becomes a fountainhead of interoperable knowledge.

### Organization

A semantic mesh is made up of folders and files like a website, but file naming and folder organization are constrained by carefully chosen conventions. 



#### Composability and Depth

Meshes can be composed into new meshes by dropping part of one into another.

Also namespaces in the mesh can be nested arbitrarily deep.

Each namespace acts as a holon — a self-contained whole that is simultaneously a part of its parent context. The overall structure forms a holarchy.

This holonic organization enables modularity, reusability, and localized control at each level of the semantic mesh.

#### Namespace Folders

At the root of any semantic mesh is a namespace.

Although a mesh can be published to the web, and so be given a fully-qualified domain name, for composability purposes the root of a namespace should be a single-term folder. Because every contained element shares this root, the fewer character the better. "ns/" is a good choice.

#### Identifier Folders

It turns out that every element in a semantic mesh has an identifier sub-folder. Even _assets elements, which are terminal, have them. 

#### SFlow Elements Folders

![[sflow.concepts.mesh.folder#folder-types]]

Any folder that directly contains a file that matches its name is considered a **dataset**; folders without root‐level distributions are treated as “things” or “namespaces.”

### Example Mesh Hierarchy

![[sflow.concepts.mesh.example]]

---

### 2. Constraints

1. **Exactly three marker dirs**
   Only `_id/` (for everything), `_ref/`, and (optionally) `_v-series/` are recognized. No additional “catalog” folders are needed.

2. **Dataset detection**
   A folder is a **dataset** if it has one or more `.trig` (or other RDF) files at its top level.

   * **Unversioned**: exactly one `<name>.trig` (no `_v-series/` subfolder).
   * **System‐versioned**: one “current” `<name>.trig` plus `_v-series/` with `vN/` subfolders.

3. **Type declarations**
   Each `<folder>/_id/<name>_id.trig` **must** include a triple declaring its kind:

   ```turtle
   <> a sf:Namespace .        # for namespace roots
   <> a sf:Thing .            # for “things”
   <> a sf:Dataset .          # for unversioned datasets
   <> a sf:VersionedDataset . # for system‐versioned datasets
   <> a sf:UserSeries .       # for user‐defined (contained) series
   ```

4. **Synchronization**
   The “current” `<name>.trig` in a versioned dataset **always** reflects the latest `_v-series/vN/<…>.trig`. Automate or lock this write step to avoid drift.

5. **Backlinks & provenance**
   Any cross‐resource backlinks go into the `_id/` graphs so that you can traverse your mesh by reading only `_id` files.

---

