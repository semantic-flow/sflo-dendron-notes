---
id: 0y6p8e594peoult03gobm94
title: flow dataset
desc: ''
updated: 1751262958851
created: 1730660543063
---

A **flow dataset** contains system-related administrative and structural metadata for every [[sflow.mesh.resource.node]]. 

Physically, it exists as a [[sflow.mesh.resource.folder._flow]] in a [[sflow.mesh.resource.folder.node]].

Mesh-specific metadata about a node's [[sflow.mesh.resource.element.v-series-dataset-series]] and corresponding [[sflow.mesh.resource.element.version-dataset]] mostly lives here too, eliminating the need to keep separate metadata in the element. 

// TODO: in named graphs?


The exception is the // TODO: should v-series catalog live under v-series (unversioned); it's append-only, very limited, seems apropriate.


## Use of _handle in _mesh

When () refer to nodes, they'll usually be talking about "the-node-as-mesh-constituent", so they'll use the node's [[sflow.mesh.resource.node-handle]] identifier