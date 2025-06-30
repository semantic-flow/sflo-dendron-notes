---
id: j15tveot7ja8tmxy7vvmpjz
title: v-series dataset
desc: ''
updated: 1751206726884
created: 1730660597230
---

This [[sflow.mesh.dataset-series]] primarily describes the historical series of checkpoints of a [[sflow.concept.dataset.versioned]]. It is not itself versioned using the [[sflow.concepts.dataset-versioning]] as that could cause an infinite recursion, but is the elemental linch-pin of it. 

It collects [[version dataset|sflow.mesh.resource.element.version-dataset]], which are ordered. 

Whether it can be included in the collection refers to is a matter of debate.

In addition to its role in [[sflow.concepts.mesh.node.dataset.versioned]] versioning, it can play the same role for [[sflow.mesh.resource.element.flow-dataset]], [[sflow.mesh.resource.node-handle]], and [[sflow.mesh.resource.element.reference-dataset]], making it the only element that can contained in other elements.

## Containment Rules

- can-be-contained-in: 
  - [[sflow.mesh.resource.node.dataset]]
  - [[sflow.mesh.resource.element.flow-dataset]]
  - [[sflow.mesh.resource.element.reference-dataset]]

- cannot-be-contained-in:
  - [[sflow.mesh.resource.node-handle]] (too small and static to justify versioning)
  - [[sflow.mesh.resource.asset-tree]] (no dataset to version)
  - [[sflow.mesh.resource.node.namespace]] : (no dataset to version)
  - [[sflow.mesh.resource.node.reference]] : (no direct dataset to version, but can be contained in a reference node's  [[sflow.mesh.resource.element.reference-dataset]])
  - other [[sflow.mesh.resource.element.version-dataset]] (preventing infinite recursion)

^ezb4mmwk9rvl