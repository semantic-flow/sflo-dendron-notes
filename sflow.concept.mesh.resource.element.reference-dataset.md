---
id: 9bu4sgma5c54bmwyl6ofg4c
title: reference datasets
desc: ''
updated: 1751435573646
created: 1751001012190
---

- one of the [[semantic flow elements|sflow.concept.mesh.resource.element]]
- used for attaching data to a [[sflow.concept.mesh.resource.node.reference]]
- logically, it's a [[sflow.concept.mesh.resource-facet.dataset.versioned]], so contains a [[sflow.concept.mesh.resource.element.v-series-dataset-series]]
- physically represented as a [[sflow.concept.mesh.resource.folder._ref]] with a [[sflow.concept.mesh.resource.folder._v-series]]
- metadata is kept in its parent node's meta dataset
- not a [[sflow.concept.mesh.resource.node]], so doesn't have a [[sflow.concept.mesh.resource.element.meta-dataset]]