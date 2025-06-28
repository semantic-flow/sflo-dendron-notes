---
id: j15tveot7ja8tmxy7vvmpjz
title: v-series dataset
desc: ''
updated: 1751088696869
created: 1730660597230
---

This [[sflow.concepts.sflow-dataset-series]] primarily describes the historical series of checkpoints of a [[sflow.concepts.sflow-dataset.versioned]]. It is not itself versioned using the [[sflow.concepts.sflow-dataset-versioning]] as that could cause an infinite recursion, but is the elemental linch-pin of it. 

It collects [[version dataset|sflow.concepts.element.version-dataset]], which are ordered. 

Whether it can be included in the collection refers to is a matter of debate.

In addition to its role in [[sflow.concepts.mesh-node.dataset.versioned]] versioning, it can play the same role for [[sflow.concepts.element.catalog-dataset]] and [[sflow.concepts.element.reference-dataset]], making it the only element that can contained in other elements.