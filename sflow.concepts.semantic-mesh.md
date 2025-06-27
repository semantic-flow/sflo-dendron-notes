---
id: h6ssv16gdyf56gg235dxv85
title: semantic mesh
desc: ''
updated: 1750973575670
created: 1750624002110
---

So, for a mesh definition:

- every thing in a mesh (mesh-nodes, mesh-elements, anything else?) is a   mesh-resource
  - mesh-resources correspond physically with either mesh-folders, distributions
    contained in datasets, resource pages (index.html files), or README.md and CHANGELOG.md files
  - resource-pages, README.md, and CHANGELOG.md may be contained in any mesh-folder, and provide context for whatever folder contains them
  - assets-catalogs are considered "in" or part of the mesh
  - other than assets-catalogs, the files and folders contained in assets-nodes are "attached" to a mesh, but not part of it.
- mesh-nodes are represented physically as folders where the (local) name of the  node is the same as the folder name
- mesh-nodes are namespace-nodes, semantic-nodes, dataset-nodes,  dataset-series-nodes and assets-nodes
- logically, mesh-folders always extend the namespace with a namespace segment  that corresponds to the node name
- semantic-nodes are always containers (or potential containers) of other  semantic-nodes and/or mesh-elements
- reserved mesh-node names are _assets
- semantic-nodes must have reference-datasets
- except for assets-nodes, mesh-nodes may contain other mesh nodes
- dataset-series-nodes can contain version-series elements, system-elements
- dataset-nodes can only contain a catalog, an optional assets-node and one  required distribution
- mesh-elements are either datasets (reference-dataset, catalog-dataset,  vseries-dataset, version-dataset) or distribution files of those datasets
- with the exception of the changelog and readme, all the  other mesh-elements are "system elements", i.e., not intended for user modification
- mesh-elements are either system-elements, distributions, resource-pages or CHANGELOG.md or README.md files; they all need containing
- system-elements 
