---
id: p3mbdrze0qe8uvko4i16t1s
title: mesh folder
desc: ''
updated: 1750711332307
created: 1750659145476
---

- a mesh is structured with mesh folders, which correspond to RDF resources. When a mesh gets published, the folders also correspond to URL paths.




### 
* **`_assets/`**
  Arbitrary sub-folders and files, to be included verbatim when publishing to a [[sflow.concepts.site]]. Typically, they will supplement or complement the semantic content in the mesh. They are terminal, i.e., they can't contain [[sflow.concepts.elements]]


## Rejected options

- directories: longer to type and most people just say folders
- URLs/IRIs: a mesh is local-first, and we don't use the "file:// schema" so it doesn't make sense to call these things URLs
- folder names are used for identifiers, but identifiers are contextualized with their containing path. if you're not talking about the whole path, it's really just a folder.