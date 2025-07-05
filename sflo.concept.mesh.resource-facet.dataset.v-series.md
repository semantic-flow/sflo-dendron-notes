---
id: j15tveot7ja8tmxy7vvmpjz
title: v-series dataset facet
desc: ''
updated: 1751690844510
created: 1730660597230
---

Datasets with the **v-series dataset facet** have a historical series of checkpoints. [[sflo.concept.mesh.resource-facet.dataset.versioned]]. It is not itself versioned using the [[sflo.concept.dataset-versioning]] as that could cause an infinite recursion, but is the elemental linch-pin of it. 

It collects [[version dataset|sflo.concept.mesh.resource.element.version-dataset]], which are ordered. 

Whether it can be included in the collection refers to is a matter of debate.

In addition to its role in [[sflo.concepts.mesh.node.dataset.versioned]] versioning, it can play the same role for [[sflo.concept.mesh.resource.element.meta-dataset]], [[sflo.concept.mesh.resource.element.node-handle]], and [[sflo.concept.mesh.resource.element.reference-dataset]], making it the only element that can contained in other elements.

## Containment Rules

- can-be-contained-in: 
  - [[sflo.concept.mesh.resource.node.data]]
  - [[sflo.concept.mesh.resource.element.meta-dataset]]
  - [[sflo.concept.mesh.resource.element.reference-dataset]]

- cannot-be-contained-in:
  - [[sflo.concept.mesh.resource.element.node-handle]] (too small and static to justify versioning)
  - [[sflo.concept.mesh.resource.element.asset-tree]] (no dataset to version)
  - [[sflo.concept.mesh.resource.node.namespace]] : (no dataset to version)
  - [[sflo.concept.mesh.resource.node.reference]] : (no direct dataset to version, but can be contained in a reference node's  [[sflo.concept.mesh.resource.element.reference-dataset]])
  - other [[sflo.concept.mesh.resource.element.version-dataset]] (preventing infinite recursion)

^ezb4mmwk9rvl