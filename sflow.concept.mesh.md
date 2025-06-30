---
id: h6ssv16gdyf56gg235dxv85
title: semantic mesh
desc: ''
updated: 1751268662460
created: 1750624002110
---

- every thing in a mesh ( [[sflow.concept.mesh.resource.node]], [[sflow.concept.mesh.resource.element]], [[sflow.concept.mesh.resource.file.resource-page]], [[sflow.concept.mesh.resource.file.changelog]], and [[sflow.concept.mesh.resource.file.readme]] ) is an addressable [[sflow.concept.mesh.resource]]
  - mesh resources correspond physically with either [[sflow.concept.mesh.resource.folder]], distributions contained in [[datasets|sflow.concept.dataset]], resource pages (index.html files), or README.md and CHANGELOG.md files
  - resource pages should be present in every [[sflow.concept.mesh.resource.folder]], at least after a [[sflow.concept.weave]]
  - README.md, and CHANGELOG.md may be contained in any [[sflow.concept.mesh.resource-type.user]], and provide introductory and historical context respectively for whatever folder contains them
  - [[sflow.concept.mesh.resource.asset-tree]] catalogs are considered "in" or part of the mesh
  - other than asset-tree catalogs, the files and folders contained in assets-trees are "attached" to a mesh, but not part of its structure.
- mesh-nodes are represented physically as folders where the (local) name of the  node is the same as the folder name
- mesh-nodes are namespace-nodes, semantic-nodes, dataset-nodes, dataset-series-nodes
- logically, mesh-folders always extend the namespace with a namespace segment  that corresponds to the folder name
- [[sflow.concept.mesh.resource.node]] are always containers (or potential containers) of other nodes, mesh elements, and 
- reserved resource names begin with an underscore
- semantic-nodes must have reference-datasets
- except for assets-nodes, mesh-nodes may contain other mesh nodes
- dataset-series-nodes can contain version-series elements, system-elements
- dataset-nodes can only contain a catalog, an optional assets-node and one  required distribution
- mesh-elements are either datasets (reference-dataset, catalog-dataset,  vseries-dataset, version-dataset) or distribution files of those datasets
- with the exception of the changelog and readme, all the  other mesh-elements are "system elements", i.e., not intended for user modification
- mesh-elements are either system-elements, distributions, resource-pages or CHANGELOG.md or README.md files; they all need containing
