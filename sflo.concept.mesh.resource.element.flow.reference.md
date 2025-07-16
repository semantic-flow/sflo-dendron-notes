---
id: 9bu4sgma5c54bmwyl6ofg4c
title: reference flow
desc: ''
updated: 1751988804979
created: 1751001012190
---

- one of the [[semantic flow elements|sflo.concept.mesh.resource.element]]
- used for attaching data to a [[sflo.concept.mesh.resource.node.reference]]
- logically, it's a [[sflo.concept.mesh.flow-facet.versioned]], so contains a [[sflo.concept.mesh.flow-facet.v-series]]
- physically represented as a [[sflo.concept.mesh.resource.folder._ref-flow]]
- not a [[sflo.concept.mesh.resource.node]], so doesn't have its own [[sflo.concept.mesh.resource.element.flow.metadata]]
  - metadata is kept in its parent node's metadata flow
