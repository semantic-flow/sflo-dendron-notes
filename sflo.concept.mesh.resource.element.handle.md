---
id: o2926clbf63vi9kln79kapg
title: node handle
desc: ''
updated: 1751749431518
created: 1751126532834
---

A **handle** is a very simple element that can only be contained in [[sflo.concept.mesh.resource.element.flow.reference]]. It has a very special semantic use: instead of refering to itself, it refers to its containing node "as a mesh resource."



## Justification

Element identifiers don't have an obvious referent other than themselves. e.g., ns/djradon/bio-dataset/_data refers only to a specific [[sflo.concept.mesh.resource.element.flow]]. 

So when they're mentioned in [[sflo.concept.mesh.resource.element.flow.metadata]], it's clear enough that their identifiers refer to them "as mesh resources."

But because the [[sflo.concept.identifier]] for a [[sflo.concept.mesh.resource.node]] refers to nothing (in the case of a namespace node) or a concept (in the case of a reference node or data node), based on the [[sflo.principle.single-referent]] principle, you should not use the node's URL to refer to it "as a mesh resource."

(note the difference between an abstract dataset, which helps define a data node; and the concept to which a data node refers.)


## Containment Rules

- must-be-contained-in: [[sflo.concept.mesh.resource.node]]
- cannot-be-contained-in: 
  - [[sflo.concept.mesh.resource.element]]
