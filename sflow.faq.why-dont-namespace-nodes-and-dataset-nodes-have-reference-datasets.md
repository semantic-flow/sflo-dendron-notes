---
id: faq001
title: Why don't namespace nodes and data nodes have reference datasets?
desc: ''
updated: 1751385453122
created: 1751351297000
---

## Question

Why don't [[namespace nodes|sflow.concept.mesh.resource.node.namespace]] and [[data nodes|sflow.concept.mesh.resource.node.data]] have [[reference datasets|sflow.concept.mesh.resource.element.reference-dataset]] like [[reference nodes|sflow.concept.mesh.resource.node.reference]] do?

## Answer

Only [[reference nodes|sflow.concept.mesh.resource.node.reference]] have reference datasets because they serve fundamentally different purposes:

### Reference Nodes
Reference nodes exist specifically to refer to external entities - people, concepts, relationships, or things that exist outside the mesh. Their [[reference datasets|sflow.concept.mesh.resource.element.reference-dataset]] contain data **about their referent** - the external thing they point to.

### data nodes  
data nodes represent **the dataset itself**. The dataset identifier should refer directly to "this dataset" as a concrete data resource, not to some external abstract concept. If you need to reference an external dataset that this one is related to, the proper approach is to:

1. Create a separate [[reference node|sflow.concept.mesh.resource.node.reference]] for that external dataset
2. Use the reference node's [[reference dataset|sflow.concept.mesh.resource.element.reference-dataset]] to describe the external dataset
3. Link the two datasets through appropriate RDF properties in either dataset

### Namespace Nodes
Namespace nodes are organizational containers that extend the URL namespace. They don't refer to external entities - they **are** the namespace segment. Adding reference datasets would create confusion about whether the namespace refers to itself or to some external concept.

## Design Principle

This maintains the [[single referent principle|sflow.principle.single-referent]]: each mesh resource has a clear, unambiguous referent. Reference nodes refer to external things, while dataset and namespace nodes refer to themselves as mesh resources.