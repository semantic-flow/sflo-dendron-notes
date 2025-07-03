---
id: 0y6p8e594peoult03gobm94
title: meta dataset
desc: ''
updated: 1751435070657
created: 1730660543063
---

A **meta dataset** contains system-related administrative and structural metadata for every [[sflo.concept.mesh.resource.node]] and the  

Physically, it exists as a [[sflo.concept.mesh.resource.folder._catalog]] in a [[sflo.concept.mesh.resource.folder.node]].

Mesh-specific metadata about a node's [[sflo.concept.mesh.resource.element.v-series-dataset-series]] and its corresponding [[sflo.concept.mesh.resource.element.version-dataset]] mostly lives here too, eliminating the need to keep separate metadata in the element. The exception is the [[sflo.concept.mesh.resource.element.v-series-dataset-series]], which is a simple 


## Use of _handle in meta datasets

When meta datasets (or any [[sflo.concept.mesh.resource-facet.system]] dataset) refer to mesh nodes, they'll usually be talking about "the-node-as-mesh-constituent", so they'll use the node's [[sflo.concept.mesh.resource.element.node-handle]] identifier