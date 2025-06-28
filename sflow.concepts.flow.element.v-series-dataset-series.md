---
id: j15tveot7ja8tmxy7vvmpjz
title: v-series dataset
desc: ''
updated: 1751138344314
created: 1730660597230
---

This [[sflow.concepts.mesh.dataset-series]] primarily describes the historical series of checkpoints of a [[sflow.concepts.mesh.dataset.versioned]]. It is not itself versioned using the [[sflow.concepts.dataset-versioning]] as that could cause an infinite recursion, but is the elemental linch-pin of it. 

It collects [[version dataset|sflow.concepts.flow.element.version-dataset]], which are ordered. 

Whether it can be included in the collection refers to is a matter of debate.

In addition to its role in [[sflow.concepts.mesh.node.dataset.versioned]] versioning, it can play the same role for [[sflow.concepts.flow.element.catalog-dataset]], [[sflow.concepts.flow.element.id-dataset]], and [[sflow.concepts.flow.element.reference-dataset]], making it the only element that can contained in other elements.

## Containment Rules

- can-be-contained-in: 
  - [[sflow.concepts.mesh.node.dataset]]
  - [[sflow.concepts.flow.repo.dataset]]
  - [[sflow.concepts.flow.element.catalog-dataset]]
  - [[sflow.concepts.flow.element.reference-dataset]]

- cannot-be-contained-in:
  - [[sflow.concepts.flow.element.id-dataset]] (too small and static to justify versioning)
  - [[sflow.concepts.flow.resource.asset-tree]] (no dataset to version)
  - [[sflow.concepts.mesh.node.namespace]] : (no dataset to version)
  - [[sflow.concepts.mesh.node.reference]] : (no direct dataset to version, but can be contained in a reference node's  [[sflow.concepts.flow.element.reference-dataset]])
  - other [[sflow.concepts.flow.element.version-dataset]] (preventing infinite recursion)

^ezb4mmwk9rvl