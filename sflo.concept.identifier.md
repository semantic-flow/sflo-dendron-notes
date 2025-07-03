---
id: 17l23vl7sqg997hr773dh23
title: Identifier
desc: ''
updated: 1751241480755
created: 1750654763700
---

## Identifier Semantics

### Node Identifier Semantics

[[sflo.concept.identifier.node]] refer to something at-least-partially abstract:

- /ns/djradon ([[sflo.concept.mesh.resource.node.reference]]) refers to a person
- /ns/djradon/bio-dataset ([[sflo.concept.mesh.resource.node.data]]) refers to a dataset in general
  - arguably this is less abstract, but say there's a dataset somewhere out there in a non-RDF format; this identifier might connote the same data as that other one
- /ns ([[sflo.concept.mesh.resource.node.namespace]]) refers to an addressable (virtual) place
  - at some point in the future, the content could change, but the address is still out there

Occasionally you'll want to refer to the nodes as "mesh-parts" instead of the things they refer to. To preserve the [[sflo.principle.single-referent]], we use the [[sflo.concept.mesh.resource.element.node-handle]]

### Element and other Resource Identifier Semantics

- arguably, these just refer to themselves, i.e., parts of a [[sflo.concept.mesh]]. For the sake of simplicity, we can consider their URLs to refer to them as "parts of a mesh".

## Identifier Name Limitations

- initial underscores prefix all reserved dataset identifiers and should be avoided in general


![[sflow#^pnoqpr3ff4za]] 