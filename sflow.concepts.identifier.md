---
id: 17l23vl7sqg997hr773dh23
title: Identifier
desc: ''
updated: 1751241480755
created: 1750654763700
---

## Identifier Semantics

### Node Identifier Semantics

[[sflow.concepts.identifier.node]] refer to something at-least-partially abstract:

- /ns/djradon ([[sflow.mesh.resource.node.reference]]) refers to a person
- /ns/djradon/bio-dataset ([[sflow.mesh.resource.node.dataset]]) refers to a dataset in general
  - arguably this is less abstract, but say there's a dataset somewhere out there in a non-RDF format; this identifier might connote the same data as that other one
- /ns ([[sflow.mesh.resource.node.namespace]]) refers to an addressable (virtual) place
  - at some point in the future, the content could change, but the address is still out there

Occasionally you'll want to refer to the nodes as "mesh-parts" instead of the things they refer to. To preserve the [[sflow.principle.single-referent]], we use the [[sflow.mesh.resource.node-handle]]

### Element and other Resource Identifier Semantics

- arguably, these just refer to themselves, i.e., parts of a [[sflow.mesh]]. For the sake of simplicity, we can consider their URLs to refer to them as "parts of a mesh".

## Identifier Name Limitations

- initial underscores prefix all reserved dataset identifiers and should be avoided in general


![[sflow#^pnoqpr3ff4za]] 