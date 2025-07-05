---
id: o2926clbf63vi9kln79kapg
title: node handle
desc: ''
updated: 1751634049304
created: 1751126532834
---

Element identifiers don't have an obvious referent other than themselves. e.g., ns/djradon/bio-dataset/_v-series refers to the [[sflo.concept.mesh.resource-facet.dataset.v-series]]. So when they're mentioned in [[sflo.concept.mesh.resource.element.meta-dataset]], it's clear enough that their identifiers refer to them "as mesh resources."

But because the [[sflo.concept.identifier]] for a [[sflo.concept.mesh.resource.node]] refers to an abstract name, dataset, or thing, based on the [[sflo.principle.single-referent]] principle, you should use the node's URL to refer to it "as a mesh resource"

A **node handle** is a very simple folder with a corresponding [[sflo.concept.mesh.resource.element.node-handle.page]]. It has a very special semantic use: instead of refering to itself, it refers to its containing node "as a mesh resource."


## Containment Rules

- must-be-contained-in: [[sflo.concept.mesh.resource.node]]
- cannot-be-contained-in: 
  - [[sflo.concept.mesh.resource.element]]
