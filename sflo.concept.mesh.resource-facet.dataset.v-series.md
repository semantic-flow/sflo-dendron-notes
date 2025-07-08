---
id: j15tveot7ja8tmxy7vvmpjz
title: v-series dataset facet
desc: ''
updated: 1751747499554
created: 1730660597230
---

Datasets with the **v-series dataset facet** have a historical series of checkpoints. [[sflo.concept.mesh.resource.element.node-component.versioned]]. It is not itself versioned using the [[sflo.concept.dataset-versioning]] as that could cause an infinite recursion, but is the elemental linch-pin of it. 

It collects [[Version|sflo.concept.mesh.resource.element.node-component.layer.version]], which are ordered. 

Whether it can be included in the collection refers to is a matter of debate.

In addition to its role in [[sflo.concepts.mesh.node.dataset.versioned]] versioning, it can play the same role for [[sflo.concept.mesh.resource.element.node-component.metadata]], [[sflo.concept.mesh.resource.element.node-handle]], and [[sflo.concept.mesh.resource.element.node-component.reference]], making it the only element that can contained in other elements.

## Containment Rules

- can-be-contained-in: 
  - [[sflo.concept.mesh.resource.element.node-component]], i.e.:
    - [[sflo.concept.mesh.resource.element.node-component.reference]]
    - [[sflo.concept.mesh.resource.element.node-component.data]]
    - [[sflo.concept.mesh.resource.element.node-component.metadata]]

- cannot-be-contained-in: everything else

^ezb4mmwk9rvl