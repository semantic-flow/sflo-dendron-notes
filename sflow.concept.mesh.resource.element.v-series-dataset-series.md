---
id: j15tveot7ja8tmxy7vvmpjz
title: v-series dataset
desc: ''
updated: 1751206726884
created: 1730660597230
---

This [[sflow.concept.mesh.resource.node.dataset.series]] primarily describes the historical series of checkpoints of a [[sflow.concept.mesh.dataset-type.versioned]]. It is not itself versioned using the [[sflow.concept.mesh.dataset-versioning]] as that could cause an infinite recursion, but is the elemental linch-pin of it. 

It collects [[version dataset|sflow.concept.mesh.resource.element.version-dataset]], which are ordered. 

Whether it can be included in the collection refers to is a matter of debate.

In addition to its role in [[sflow.concepts.mesh.node.dataset.versioned]] versioning, it can play the same role for [[sflow.concept.mesh.resource.element.catalog-dataset]], [[sflow.concept.mesh.resource.element.node-handle]], and [[sflow.concept.mesh.resource.element.reference-dataset]], making it the only element that can contained in other elements.

## Containment Rules

- can-be-contained-in: 
  - [[sflow.concept.mesh.resource.node.dataset]]
  - [[sflow.concept.mesh.resource.element.catalog-dataset]]
  - [[sflow.concept.mesh.resource.element.reference-dataset]]

- cannot-be-contained-in:
  - [[sflow.concept.mesh.resource.element.node-handle]] (too small and static to justify versioning)
  - [[sflow.concept.mesh.resource.asset-tree]] (no dataset to version)
  - [[sflow.concept.mesh.resource.node.namespace]] : (no dataset to version)
  - [[sflow.concept.mesh.resource.node.reference]] : (no direct dataset to version, but can be contained in a reference node's  [[sflow.concept.mesh.resource.element.reference-dataset]])
  - other [[sflow.concept.mesh.resource.element.version-dataset]] (preventing infinite recursion)

^ezb4mmwk9rvl