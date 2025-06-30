---
id: 0y6p8e594peoult03gobm94
title: mesh dataset
desc: ''
updated: 1751239858053
created: 1730660543063
---

A **mesh dataset** contains system-related administrative and structural metadata for every [[sflow.mesh.resource.node]]. 

Physically, it exists as a [[sflow.mesh.resource.folder._flow]] in a [[sflow.mesh.resource.folder.node]].

Metadata about a node's [[sflow.mesh.resource.element.v-series-dataset-series]] and corresponding [[sflow.mesh.resource.element.version-dataset]]lives here too, eliminating the need to keep separate metadata in the element.

// TODO: in named graphs?

## Use of _handle in _mesh

When mesh-dataset refer to nodes, they'll usually be talking about "the-node-as-mesh-constituent", so they'll use the node's 